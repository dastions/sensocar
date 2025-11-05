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
