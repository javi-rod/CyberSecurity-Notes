# Identificación, Autenticación, Autorización y Accountability

Este archivo detalla el flujo técnico desde que un usuario reclama una identidad hasta que sus acciones son auditadas en un sistema profesional.

## Identificación

Es el acto mediante el cual un sujeto (usuario, proceso o sistema) reclama una identidad única.

* **Identificadores comunes**: 

    * Nombres de usuario (ej. `tanderson`, `neo`).
    
    * Números únicos (ID de estudiante, Pasaporte, número de móvil).
    
    * Correo electrónico (muy usado por su garantía de ser único en internet).

* **Concepto clave**: La identificación por sí sola no requiere pruebas; es simplemente decir "quién eres". La verificación viene en el siguiente paso.

## Autenticación

Es el proceso de verificar la identidad reclamada. Se basa en tres mecanismos principales y uno combinado:

### 1. Algo que sabes (Something you know)

Cosas que el usuario memoriza.

* **Ejemplos**: Contraseñas, frases de contraseña (passphrases) o códigos PIN.

### 2. Algo que tienes (Something you have)

Objetos físicos o digitales en posesión del usuario.

* **Ejemplos**: Teléfono móvil (para recibir SMS), llaves de seguridad de hardware (Yubico, Titan, Nitrokey) o generadores de OTP.

### 3. Algo que eres (Something you are)

Lectores biométricos basados en características físicas.

* **Ejemplos**: Huellas dactilares, reconocimiento facial, escáneres de retina o reconocimiento de voz.


### Multi-Factor Authentication (MFA / 2FA)

Uso de **dos o más** de los mecanismos anteriores. 

* **Importancia**: Proporciona seguridad adicional si uno de los factores (como la contraseña) se ve comprometido. Un ejemplo clásico es el cajero automático (Tarjeta + PIN).

## Autorización y Control de Acceso

* **Autorización**: Decide **qué** tiene permitido hacer el usuario (ej. leer un archivo pero no borrarlo).

* **Control de Acceso**: Es el mecanismo técnico que **ejecuta** esa decisión (ej. el sistema de archivos denegando el borrado).

* **Principio de Seguridad**: Se debe restringir el acceso solo a los recursos necesarios para que el usuario realice sus funciones.

## Accountability, Logging y SIEM/SOAR

Asegura que los usuarios sean responsables de sus acciones mediante la trazabilidad.

### Logging (Registro)

Proceso de grabar eventos (quién, qué, cuándo). Es vital para:

* Cumplimiento normativo (Compliance).

* Respuesta ante incidentes.

* Investigaciones forenses.

### Gestión Avanzada: SIEM y SOAR

Para manejar el volumen masivo de logs en empresas, se utilizan:

* **SIEM (Security Information and Event Management)**: Se centra en la **Detección**. Recolecta logs de toda la red, los correlaciona y genera alertas si detecta algo sospechoso.

* **SOAR (Security Orchestration, Automation, and Response)**: Se centra en la **Respuesta**. Permite crear "playbooks" o flujos automáticos para reaccionar a las alertas sin intervención humana (ej. bloquear una IP en el firewall automáticamente).