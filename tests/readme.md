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
