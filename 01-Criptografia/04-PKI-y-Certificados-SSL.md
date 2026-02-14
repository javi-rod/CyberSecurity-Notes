# PKI (Public Key Infrastructure) y SSL/TLS

## PKI (Public Key Infrastructure)

Como vimos en Diffie-Hellman, el cifrado no sirve de nada si no sabemos con quién estamos hablando. Un atacante (**Man-in-the-Middle**) podría interceptar la comunicación. La **Infraestructura de Clave Pública (PKI)** soluciona esto vinculando identidades con claves públicas mediante certificados.

---

## Elementos Clave de PKI

- **Certificado Digital:** Un documento electrónico que vincula una clave pública con una identidad (ej. un dominio como `google.com`).

- **CA (Certificate Authority):** El "Notario" digital (ej. DigiCert, Let's Encrypt). Si tu navegador confía en la CA, confía en los certificados que ella firma.

- **CSR (Certificate Signing Request):** El archivo que generas para pedirle a una CA que firme tu clave pública.

- **CRL / OCSP:** Listas y protocolos para verificar si un certificado ha sido revocado antes de su fecha de expiración.

---

## Comandos Prácticos (OpenSSL)

### 1. Crear una solicitud de firma (CSR)

Este comando genera una **clave privada** nueva y el archivo `.csr` que enviarías a una CA para que lo firme.

```bash
openssl req -new -nodes -newkey rsa:4096 -keyout server.key -out server.csr
```
### 2. Crear un certificado Autofirmado (Self-Signed)

Ideal para laboratorios o pruebas internas donde no necesitas una CA externa.

```bash
openssl req -x509 -newkey rsa:4096 -nodes -keyout key.pem -out cert.pem -sha256 -days 365
```

- `-x509`: Indica que queremos el certificado final (el "DNI" ya emitido).

- `-days 365`: Define la validez (importante para evitar alertas de expiración).

### 3. Inspeccionar un Certificado (Ver propiedades)

Para verificar quién emitió un certificado o su fecha de caducidad:

```bash
openssl x509 -in cert.pem -text -noout
```
---


## El protocolo SSL/TLS

Aunque solemos decir "SSL", hoy en día usamos TLS (Transport Layer Security). Es el protocolo que usa PKI para establecer una conexión segura.

- **Handshake**: El cliente y el servidor se saludan y acuerdan qué algoritmos usar.

- **Autenticación**: El servidor envía su certificado. El cliente verifica con su lista de CAs confiables.

- **Intercambio de Clave**: Usan Diffie-Hellman o RSA para generar una clave simétrica temporal.

- **Cifrado de Sesión**: Todos los datos viajan con cifrado simétrico (más rápido).