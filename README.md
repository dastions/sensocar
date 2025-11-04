# Sensocar - Guía de Instalación y Uso

Este documento proporciona instrucciones completas para instalar y configurar el servicio Sensocar en una Raspberry Pi.

## 📋 Tabla de Contenidos

- [Requisitos](#requisitos)
- [Instalación de Docker en Raspberry Pi](#instalación-de-docker-en-raspberry-pi)
- [Configuración del Entorno](#configuración-del-entorno)
- [Docker Compose](#docker-compose)
- [API del Servicio](#api-del-servicio)
- [API Externa (SENSOCAR_SERVER)](#api-externa-sensocar_server)
- [Prueba del Servidor](#prueba-del-servidor)
- [Solución de Problemas](#solución-de-problemas)
- [Licencia](#licencia)

## Requisitos

- Raspberry Pi con Raspberry Pi OS (preferiblemente la versión más reciente)
- Acceso a internet para descargar dependencias
- Acceso SSH a la Raspberry Pi (opcional pero recomendado)
- Credenciales de acceso proporcionadas por Dastions

## Instalación de Docker en Raspberry Pi

### Paso 1: Actualizar el sistema

```bash
sudo apt update
sudo apt upgrade -y
```

### Paso 2: Instalar dependencias

```bash
sudo apt install -y \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg \
    lsb-release
```

### Paso 3: Agregar la clave GPG oficial de Docker

```bash
curl -fsSL https://download.docker.com/linux/debian/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
```

### Paso 4: Configurar el repositorio estable

```bash
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/debian \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

### Paso 5: Instalar Docker Engine

```bash
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-compose-plugin
```

### Paso 6: Agregar tu usuario al grupo docker (opcional, para evitar usar sudo)

```bash
sudo usermod -aG docker $USER
```

**Nota:** Necesitarás cerrar sesión y volver a iniciar sesión para que este cambio surta efecto.

### Paso 7: Verificar la instalación

```bash
docker --version
docker compose version
```

Deberías ver las versiones instaladas de Docker y Docker Compose.

## Configuración del Entorno

### Paso 1: Crear el archivo `.env`

Crea un archivo `.env` en el directorio raíz del proyecto con las siguientes variables:

```env
# Variables requeridas - Solicitar a support@dastions.com
SENSOCAR_SERVER_API=
SENSOCAR_API_KEY=
SERVER_API=https://staging.appsmartpay.com
DEVICE_API_KEY=
DEVICE_TOKEN=

# Configuración del servidor
HTTP_PORT=3010
TCP_PORT=3011

# Configuración OCR (si aplica)
OCR_SERVER_API=https://ocr.dastions.com
OCR_API_KEY=DASTIONS.OCR.123

# Configuración de fotos (si aplica)
PHOTO_SERVER_API=https://blob.dastions.com
PHOTO_API_KEY=

# Identificación del dispositivo
DEVICE_ID=M2
DEVICE_NAME=Margalef_2

# Configuración adicional
INTERNET_CHECK=true
DEVICE_DATA_INTERVAL=30000
JWT_SECRET_KEY=

# Configuración de lectores (ajustar según tu configuración)
LECTOR_NAME_1=QR Reader IN
LECTOR_IP_1=192.168.10.22
LECTOR_PORT_1=4001

KEYPAD_NAME_1=KeyPad Reader IN
KEYPAD_IP_1=192.168.10.23
KEYPAD_PORT_1=4001

LECTOR_NAME_2=QR Reader OUT
LECTOR_IP_2=192.168.10.24
LECTOR_PORT_2=4001

KEYPAD_NAME_2=KeyPad Reader OUT
KEYPAD_IP_2=192.168.10.21
KEYPAD_PORT_2=4001

# Configuración DIO (si aplica)
DIO_IP=192.168.10.25
DIO_PORT=80
DIO_USER=admin
DIO_PASSWORD=
BARRERA_RELAY=0
BARRERA_TIMEOUT=6000
BARRERA_ENABLED=true

# Configuración de cámaras (si aplica)
CAMERA_ID_1=CAMERA_IN
CAMERA_MODEL_1=axis
CAMERA_IP_1=192.168.1.5
CAMERA_PORT_1=80
CAMERA_AUTO_1=false
CAMERA_USER_1=admin
CAMERA_PASSWORD_1=

CAMERA_ID_2=CAMERA_OUT
CAMERA_MODEL_2=dahua
CAMERA_IP_2=192.168.1.5
CAMERA_PORT_2=80
CAMERA_AUTO_2=false
CAMERA_USER_2=admin
CAMERA_PASSWORD_2=

# Debug (opcional)
KEYPAD_DEBUB=false
LECTOR_DEBUB=false
DIO_DEBUG=false
```

### Paso 2: Solicitar credenciales

Las siguientes variables son **requeridas** y deben solicitarse a **support@dastions.com**:

- `SENSOCAR_SERVER_API` - URL del servidor Sensocar
- `SENSOCAR_API_KEY` - Clave de API para autenticación
- `SERVER_API` - URL del servidor (por defecto: `https://staging.appsmartpay.com`)
- `DEVICE_API_KEY` - Clave de API del dispositivo
- `DEVICE_TOKEN` - Token de autenticación del dispositivo

## Docker Compose

### Archivo docker-compose.yml

Crea un archivo `docker-compose.yml` con el siguiente contenido:

```yaml
services:
  sensocar:
    image: ghcr.io/dastions/sensocar:latest
    restart: always
    ports:
      - target: 4001
        published: 4001
        protocol: tcp
        mode: host
      - target: 3000
        published: 3010
        protocol: tcp
        mode: host
    extra_hosts:
      - host.docker.internal:host-gateway
    env_file:
      - .env
    environment:
      - TCP_PORT=3001
      - HTTP_PORT=3000
      - SERVER_API=http://host.docker.internal:2001
      - OCR_SERVER_API=https://ocr.dastions.com
```

**Nota:** Asegúrate de que el archivo `.env` esté en el mismo directorio que el `docker-compose.yml`.

### Ejecutar el servicio

Para iniciar el servicio:

```bash
docker compose up -d
```

Para ver los logs:

```bash
docker compose logs -f
```

Para detener el servicio:

```bash
docker compose down
```

Para reiniciar el servicio:

```bash
docker compose restart
```

## API del Servicio

El servicio expone los siguientes endpoints:

### POST `/api/sensocar/lector`

Registra una lectura del lector QR/tarjeta.

**Request Body:**
```json
{
  "code": "string"
}
```

**Response:**
```json
{
  "data": "ok"
}
```

**Ejemplo con curl:**
```bash
curl -X POST http://localhost:3010/api/sensocar/lector \
  -H "Content-Type: application/json" \
  -d '{"code": "123456789"}'
```

### POST `/api/sensocar/weight`

Registra el peso detectado por la báscula.

**Request Body:**
```json
{
  "weight": 1250
}
```

**Response:**
```json
{
  "data": "ok"
}
```

**Ejemplo con curl:**
```bash
curl -X POST http://localhost:3010/api/sensocar/weight \
  -H "Content-Type: application/json" \
  -d '{"weight": 1250}'
```

**Nota:** Si el peso es mayor a 100 y el estado actual es 0, el sistema cambiará el estado a 1.

## API Externa (SENSOCAR_SERVER)

El servicio utiliza las siguientes APIs para comunicarse con el servidor Sensocar:

### GET `/weight`

Obtiene el peso actual del servidor Sensocar.

**Headers:**
```
Authorization: Bearer {SENSOCAR_API_KEY}
```

**Response:**
```json
{
  // Respuesta del servidor Sensocar
}
```

### POST `/display`

Envía un mensaje para mostrar en el display del dispositivo Sensocar.

**Headers:**
```
Authorization: Bearer {SENSOCAR_API_KEY}
```

**Request Body:**
```json
{
  "message": "Texto a mostrar"
}
```

**Response:**
```json
{
  // Respuesta del servidor Sensocar
}
```

### POST `/payment/success`

Notifica al servidor Sensocar que un pago se completó exitosamente.

**Headers:**
```
Authorization: Bearer {SENSOCAR_API_KEY}
```

**Request Body:**
```json
{
  "token": "payment_token",
  "payment": {
    // Datos del pago
  }
}
```

**Response:**
```json
{
  // Respuesta del servidor Sensocar
}
```

### POST `/payment/error`

Notifica al servidor Sensocar que ocurrió un error en el pago.

**Headers:**
```
Authorization: Bearer {SENSOCAR_API_KEY}
```

**Request Body:**
```json
{
  "token": "payment_token",
  "payment": {
    // Datos del pago
  }
}
```

**Response:**
```json
{
  // Respuesta del servidor Sensocar
}
```

## Prueba del Servidor

### Paso 1: Verificar que el contenedor está corriendo

```bash
docker compose ps
```

Deberías ver el servicio `sensocar` con estado `Up`.

### Paso 2: Verificar los logs

```bash
docker compose logs sensocar
```

Busca mensajes que indiquen que el servicio está corriendo correctamente:
- `> Dastions Box running on port 3010`
- `> Socket.io running on port 4001`

### Paso 3: Probar el endpoint de salud

```bash
curl http://localhost:3010/api/sensocar/lector \
  -X POST \
  -H "Content-Type: application/json" \
  -d '{"code": "test123"}'
```

Deberías recibir una respuesta `{"data": "ok"}`.

### Paso 4: Probar el endpoint de peso

```bash
curl http://localhost:3010/api/sensocar/weight \
  -X POST \
  -H "Content-Type: application/json" \
  -d '{"weight": 500}'
```

Deberías recibir una respuesta `{"data": "ok"}`.

### Paso 5: Verificar conectividad con el servidor Sensocar

Revisa los logs para asegurarte de que no hay errores de conexión con `SENSOCAR_SERVER_API`:

```bash
docker compose logs -f sensocar | grep -i "sensocar\|error"
```

## Solución de Problemas

### El contenedor no inicia

1. Verifica que todas las variables de entorno requeridas estén configuradas en el archivo `.env`
2. Verifica los logs: `docker compose logs sensocar`
3. Verifica que los puertos 3010 y 4001 no estén siendo usados por otro proceso:
   ```bash
   sudo netstat -tulpn | grep -E '3010|4001'
   ```

### Error de conexión con SENSOCAR_SERVER_API

1. Verifica que `SENSOCAR_SERVER_API` esté correctamente configurado en el archivo `.env`
2. Verifica que `SENSOCAR_API_KEY` sea válido
3. Verifica la conectividad de red desde la Raspberry Pi:
   ```bash
   curl -H "Authorization: Bearer {SENSOCAR_API_KEY}" {SENSOCAR_SERVER_API}/weight
   ```

### El servicio no responde a las peticiones

1. Verifica que el servicio esté corriendo: `docker compose ps`
2. Verifica los logs para errores: `docker compose logs -f sensocar`
3. Verifica que el firewall no esté bloqueando los puertos:
   ```bash
   sudo ufw status
   ```

### Docker no está disponible después de la instalación

Si después de agregar tu usuario al grupo docker no puedes ejecutar comandos sin `sudo`:

1. Cierra sesión completamente y vuelve a iniciar sesión
2. O reinicia la Raspberry Pi: `sudo reboot`

## Licencia

Licencia: Dastions.com con Monedero Sensocar

Para más información, contacta con **support@dastions.com**

