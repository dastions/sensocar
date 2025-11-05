# Sensocar - Guía de Instalación y Uso

Este documento proporciona instrucciones completas para instalar y configurar el servicio Sensocar en una Raspberry Pi.

## 📋 Tabla de Contenidos

- [Requisitos](#requisitos)
- [Instalación de Docker en Raspberry Pi](#instalación-de-docker-en-raspberry-pi)
- [Configuración del Entorno](#configuración-del-entorno)
- [Docker Compose](#docker-compose)
- [Licencia](#licencia)

## Requisitos

- Raspberry Pi con Raspberry Pi OS (preferiblemente la versión más reciente)
- Acceso a internet para descargar dependencias
- Acceso SSH a la Raspberry Pi (opcional pero recomendado)
- Credenciales de acceso a SmarPay proporcionadas por Dastions

### Creación de Usuario
- [Entorno de pruebas](https://dastions.z28.web.core.windows.net/users/login)
- [Entorno de producción](https://app.appsmartpay.com/users/login)

### Solicitar Rol adminitrativo:
Una vez creado el usuario, **solicita a dastions** que te suban a role administrador de dispositivos y te añadan tu monedero.


# Only Developers

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
SENSOCAR_SERVER_API=<server-api>
SENSOCAR_API_KEY=<api-key>
SERVER_API=https://staging.appsmartpay.com
DEVICE_API_KEY=<api-key>
DEVICE_TOKEN=<api-token>

# Configuración del servidor
HTTP_PORT=3010
TCP_PORT=3011

# Configuración de fotos (si aplica)
PHOTO_SERVER_API=https://blob.dastions.com
PHOTO_API_KEY=

# Identificación del dispositivo
DEVICE_ID=BP_M01
DEVICE_NAME=Bascula_Sensocar_1

# Configuración adicional
INTERNET_CHECK=true
DEVICE_DATA_INTERVAL=30000
JWT_SECRET_KEY=
```

### Paso 2: Solicitar credenciales

Las siguientes variables son **requeridas** y deben solicitarse a **support@dastions.com**:

- `SENSOCAR_SERVER_API` - URL del servidor local Sensocar (`http://host.docker.internal:2001`)
- `SENSOCAR_API_KEY` - Clave de API para autenticación *si es necesaria*
- `SERVER_API` - URL del servidor (por defecto: `https://staging.appsmartpay.com`)
- `DEVICE_API_KEY` - Clave de API del dispositivo (support@dastions.com)
- `DEVICE_TOKEN` - Token de autenticación del dispositivo (support@dastions.com)

  

## Arrancar el servicio

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
      - SENSOCAR_SERVER_API=http://host.docker.internal:2001
```

**Nota:** Asegúrate de que el archivo `.env` esté en el mismo directorio que el `docker-compose.yml`.

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

## Licencia

Licencia: Dastions.com con Monedero Sensocar

Para más información, contacta con **support@dastions.com**

