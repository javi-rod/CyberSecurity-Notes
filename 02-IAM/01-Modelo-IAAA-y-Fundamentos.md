# Modelo IAAA y Fundamentos

La gestión de identidades y accesos (IAM) aborda preguntas críticas sobre cómo identificar usuarios, prevenir suplantaciones, decidir qué recursos pueden tocar y rastrear sus acciones para mantener la responsabilidad.

## El Modelo IAAA

Este modelo se compone de cuatro pilares esenciales que garantizan la **confidencialidad**, **integridad** y **disponibilidad** de la información.

### Las 4 Etapas del Modelo

| Etapa | Definición Técnica | Detalle Clave |
| :--- | :--- | :--- |
| **Identification** (Identificación) | El proceso donde el usuario reclama una identidad específica ante el sistema. | Utiliza identificadores únicos como el email, nombre de usuario o un número de ID. |
| **Authentication** (Autenticación) | El proceso de confirmar que el usuario es realmente quien dice ser. | Se logra mediante contraseñas o métodos adicionales como códigos enviados al móvil. |
| **Authorisation** (Autorización) | Determina a qué recursos u operaciones específicas tiene permitido acceder el usuario. | Se basa en asignar roles y permisos según la función laboral o nivel de seguridad. |
| **Accountability** (Responsabilidad) | Rastreo de la actividad para asegurar que el usuario sea responsable de sus actos. | Se implementa mediante el registro (logging) de actividades en una ubicación centralizada. |

---