# Enumeración de Active Directory (AD)

Esta sección es la continuación directa de la sección [Vectores de Acceso y Credenciales](./01-Vectores-de-Acceso-y-Credenciales.md). Tras obtener las primeras credenciales de Active Directory (AD), se inicia un proceso cíclico de enumeración y explotación donde cada movimiento lateral o escalada de privilegios requiere reevaluar el entorno desde la nueva posición. El objetivo es dominar la extracción de información crítica mediante metodologías situacionales, utilizando herramientas tanto nativas como externas: complementos de la MMC, comandos net, cmdlets de PowerShell-RSAT y el análisis de relaciones con BloodHound.

## Inyección de Credenciales

Para realizar una enumeración profunda en **Active Directory (AD)**, es esencial mimetizarse con el entorno utilizando herramientas nativas de Windows. Cuando se poseen credenciales válidas (`usuario:contraseña`) pero no se tiene una máquina unida al dominio, se utiliza la técnica de **inyección de credenciales en memoria**.

### Uso de Runas.exe

El binario legítimo `runas.exe` permite ejecutar procesos con privilegios de red distintos a los locales:

- **Comando clave:** `runas.exe /netonly /user:<dominio>\<usuario> cmd.exe`

- **Parámetro `/netonly`:** Crucial para máquinas que no pertenecen al dominio. Carga las credenciales solo para autenticación de red, permitiendo que cualquier herramienta lanzada desde esa nueva consola use la identidad del dominio en la red.

- **Validación:** La forma más efectiva de verificar la inyección es listar el directorio **SYSVOL** (`dir \\<FQDN>\SYSVOL\`), accesible para cualquier usuario autenticado.

### Configuración de DNS y Autenticación

- **Importancia del DNS:** Para que la resolución de nombres funcione, es necesario apuntar el servidor DNS a la IP del Controlador de Dominio (DC) mediante PowerShell (`Set-DnsClientServerAddress`).

- **Hostname vs IP:** Usar el **hostname (FQDN)** fuerza la autenticación **Kerberos**, mientras que usar la **IP** fuerza **NTLM**. Conocer esta diferencia es vital para la evasión de defensas (stealth) en ejercicios de Red Team.

- **Alcance:** Una vez inyectadas las credenciales, aplicaciones como SQL Management Studio o navegadores web utilizarán automáticamente la identidad del dominio para la autenticación de red (Windows/NTLM/Kerberos).

## Enumeración con Microsoft Management Console (MMC) y RSAT

Para una enumeración visual y holística del dominio, se utilizan los **Remote Server Administration Tools (RSAT)** a través de la **MMC**. Esta metodología es ideal para mapear la estructura de Unidades Organizativas (OU), usuarios y equipos de forma gráfica.

### Configuración y Acceso

Dado que la máquina de ataque no suele estar unida al dominio, es imprescindible lanzar la consola desde el proceso de **Runas** creado anteriormente. Esto garantiza que la MMC utilice las credenciales inyectadas para autenticarse contra el Controlador de Dominio.

1. **Ejecución:** Desde la consola de `runas.exe`, ejecutar `mmc.exe`.

2. **Complementos (Snap-ins):** Se deben añadir los tres complementos de Active Directory:

    - *Usuarios y equipos de Active Directory.*

    - *Dominios y confianzas de Active Directory.*

    - *Sitios y servicios de Active Directory.*

3. **Conexión:** Es necesario cambiar el foco de cada complemento al dominio objetivo (ej. dominio.local o el FQDN del cliente) y activar las **Características avanzadas** en el menú *Ver* para exponer todos los atributos de los objetos.

### Capacidades de Enumeración

- **Estructura de OU:** Permite visualizar cómo está organizada la empresa (ej. departamentos en la carpeta `People`).

- **Atributos de Usuario:** Inspección detallada de propiedades, descripciones y pertenencia a grupos.

- **Localización de Hosts:** Identificación de servidores y estaciones de trabajo unidas al dominio en sus respectivas carpetas.

- **Búsqueda:** Herramienta rápida para localizar objetos específicos dentro de todo el bosque.

### Análisis Comparativo

| Ventajas | Desventajas |
| :--- | :--- |
| Visión global y gráfica del entorno. | Requiere acceso por GUI (RDP/Escritorio). |
| Búsqueda rápida de objetos individuales. | No permite extraer atributos masivos de todo el dominio. |
| Permite edición directa si hay privilegios. | Menos sigiloso que las herramientas de línea de comandos. |

## Enumeración mediante Símbolo del Sistema (CMD)

El uso de comandos nativos en **CMD** es una técnica de enumeración "rápida y sucia", ideal cuando no se dispone de una interfaz gráfica (GUI), se opera a través de un **RAT** (Remote Access Trojan) o se busca evitar la detección de herramientas de seguridad más ruidosas. El comando principal para esta tarea es `net`.

### Comandos Esenciales de Enumeración

El modificador `/domain` es crítico, ya que indica al sistema que la consulta debe procesarse en un **Controlador de Dominio (DC)** y no localmente.

- **Usuarios:**

  - `net user /domain`: Lista todos los usuarios del dominio (útil para dimensionar el objetivo).

  - `net user <username> /domain`: Muestra detalles de una cuenta específica (comentarios, última sesión, caducidad de contraseña y membresía a grupos). *Nota: Solo muestra hasta 10 grupos.*

- **Grupos:**

  - `net group /domain`: Lista todos los grupos del dominio.

  - `net group "<nombre_grupo>" /domain`: Muestra los miembros de un grupo específico (ej. "Tier 1 Admins").

- **Política de Cuentas:**

  - `net accounts /domain`: Extrae la política de contraseñas (longitud mínima, historial y, fundamentalmente, el **umbral de bloqueo de cuenta**).

### Aplicación en Red Team

La información obtenida mediante `net accounts` es vital para diseñar ataques de **Password Spraying**. Conocer el umbral de bloqueo permite determinar cuántos intentos se pueden realizar por cuenta sin alertar a los administradores o denegar el servicio a los usuarios.

### Análisis Comparativo

| Ventajas | Desventajas |
| :--- | :--- |
| No requiere herramientas externas; difícil de detectar por el Blue Team. | **Obligatorio:** Debe ejecutarse desde una máquina ya unida al dominio (**Domain-joined**). |
| Compatible con scripts iniciales, macros y payloads de phishing. | Salida de datos limitada (ej. truncado de listas de grupos extensas). |
| Funciona perfectamente en entornos sin acceso a escritorio remoto. | No proporciona una visión relacional de los objetos. |

## Enumeración avanzada con PowerShell

PowerShell es la evolución del símbolo del sistema y ofrece un control granular sobre **Active Directory** mediante el uso de **cmdlets** (clases .NET). Al haber instalado las herramientas **RSAT**, disponemos de más de 50 comandos especializados que superan las limitaciones de la herramienta `net`.

### Uso de Cmdlets en Entornos No-Unidos (Non-Domain Joined)

A diferencia de `net`, los cmdlets de PowerShell permiten especificar el servidor objetivo, lo que facilita la enumeración desde una máquina atacante (usando `runas.exe`):

- **Usuarios (`Get-ADUser`):**

  - Permite extraer **todas** las propiedades de una cuenta (`-Properties *`).

  - Soporta filtros complejos (ej. `Get-ADUser -Filter 'Name -like "*stevens"'`).

- **Grupos (`Get-ADGroup` / `Get-ADGroupMember`):**

  - Resuelve el problema de la herramienta `net` al listar **todos** los miembros de un grupo, sin importar cuántos sean.

- **Objetos Genéricos (`Get-ADObject`):**

  - Permite búsquedas transversales.

  - Ejemplo crítico: identificar cuentas con `badPwdCount > 0` para excluirlas de un ataque de **Password Spraying** y evitar bloqueos accidentales.

- **Dominio (`Get-ADDomain`):**

  - Muestra contenedores de equipos, controladores de dominio y nombres distinguidos (DN).

### Acciones de Modificación

Aunque el enfoque es la enumeración, PowerShell permite realizar cambios directos si se tienen privilegios. Un ejemplo es el cambio forzado de credenciales:
`Set-ADAccountPassword -Identity <user> -Server <DC> -OldPassword <old> -NewPassword <new>`

### Análisis Comparativo

| Ventajas | Desventajas |
| :--- | :--- |
| **Poder:** Extrae muchísima más información que los comandos `net`. | **Visibilidad:** Las actividades de PowerShell son monitorizadas de cerca por los Blue Teams (Logging de bloques de script). |
| **Flexibilidad:** Permite apuntar a un servidor específico (`-Server`) sin estar unido al dominio. | **Dependencia:** Requiere la instalación previa de RSAT o cargar módulos externos (como PowerView). |
| **Automatización:** Los resultados se pueden filtrar, ordenar y exportar fácilmente (`Format-Table`). | Puede ser más ruidoso si no se ofuscan los comandos. |

## Enumeración con BloodHound

BloodHound utiliza el **pensamiento basado en grafos** para visualizar relaciones complejas y caminos de ataque que conectan nodos aparentemente aislados. El ecosistema se divide en el **Colector** (extrae datos) y la **Interfaz** (visualiza los datos en una DB Neo4j).

### Metodología de Recolección (Colectores)

Dependiendo del escenario y del sistema operativo desde el que operemos, existen dos herramientas principales para la ingesta de datos:

#### 1. SharpHound (Ejecución en Windows)

Es el colector oficial y más completo. Se recomienda para obtener el máximo detalle de sesiones activas.

- **Formatos:** `.exe` (binario) o `.ps1` (PowerShell para ejecución en memoria).

- **Uso común:** `SharpHound.exe --CollectionMethods All --Domain <dominio.local> --ExcludeDCs`

- **Ventaja:** Mayor precisión en la recolección de sesiones mediante llamadas RPC/SAMR.

#### 2. BloodHound.py (Ejecución Remota desde Linux)

Ideal para auditorías externas o cuando no se desea ejecutar código en el *target*.

- **Uso común:** `bloodhound-python -u <usuario> -p <password> -d <dominio.local> -dc <IP_del_DC> -c All`

- **Ventaja:** No activa alertas de ejecución de binarios sospechosos en el endpoint; utiliza tráfico LDAP estándar hacia el controlador de dominio.

### Estrategia de Ingesta y Análisis

Para evitar detecciones y optimizar el tiempo en una auditoría real:

1. **Captura Inicial:** Ejecutar una recolección completa (`All`) al inicio para mapear la estructura de OUs y Grupos.

2. **Actualización de Sesiones:** Las sesiones cambian constantemente. Se recomienda recolectar solo el método `Session` en horas de alta actividad (ej. 10:00 y 14:00) para capturar nuevos vectores de movimiento lateral.

3. **Análisis de Caminos (Edges):**

- **Membresías:** Identifica grupos anidados que otorgan privilegios de Admin de forma indirecta.

- **ACLs y Control:** Revela permisos para modificar objetos (ej. reset de passwords de un admin).

### Análisis Comparativo de BloodHound

| Ventajas | Desventajas |
| :--- | :--- |
| **Rutas de ataque:** Revela saltos complejos y relaciones de control (ACLs) invisibles manualmente. | **Ruido de Red:** Genera un tráfico LDAP masivo hacia el DC que es fácilmente detectable por un SIEM. |
| **Análisis Offline:** Permite exportar los datos (JSON) y estudiarlos sin conexión activa al objetivo. | **Snapshot Temporal:** Los datos de sesiones (LogOns) caducan rápido; si el usuario desconecta, la ruta se pierde. |
| **Mapeo de ACLs:** Expone permisos como `GenericAll` o `WriteDacl` para tomar control de cuentas. | **Firma de Ingestores:** SharpHound es un binario altamente "quemado" y vigilado por soluciones de seguridad. |

### Recursos de Instalación y Documentación

Para configurar el entorno de análisis (GUI y Base de Datos) en sistemas **Linux**, se recomienda seguir la guía oficial de **Kali Tools**, que es la referencia más completa para gestionar las dependencias de `neo4j` y las versiones de Java.

- **Guía de Instalación en Kali:** [BloodHound - Kali Linux Tools](https://www.kali.org/tools/bloodhound/)

- **Repositorio del Ingestor (Python):** [BloodHound.py en GitHub](https://github.com/dirkjanm/bloodhound.py)

> **Nota técnica:** Es vital asegurar la compatibilidad entre la versión del colector (SharpHound/BloodHound.py) y la versión de la interfaz de BloodHound instalada para evitar errores durante la ingesta de archivos JSON.
