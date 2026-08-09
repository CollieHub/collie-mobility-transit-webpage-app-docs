# 🔄 Pipeline de Sincronización (AWS Backoffice ➔ Cloudflare D1)

Instrucciones del flujo de publicación entre la consola interna de AWS y la base de datos pública Cloudflare D1.

---

## 📌 Mecanismo de Sincronización

1. **Edición Interna en AWS**:
   - El administrador edita líneas, paradas u horarios en la consola/panel de AWS Backoffice.
2. **Acción de Publicación ("Publish Catalog")**:
   - Al presionar **Publicar** en AWS Admin, una función Lambda privada compila los registros modificados.
3. **Webhook Seguro de Push a Cloudflare**:
   - La Lambda en AWS realiza una llamada HTTP POST segura firmada con API Key de servicio a `https://public-api.pordondeviene.ar/v1/internal/sync-catalog`.
4. **Actualización Atómica en Cloudflare D1**:
   - El backend Hono en Cloudflare inserta/actualiza las tablas en Cloudflare D1 de forma transaccional (`db.batch(...)`).

---

## 🛡️ Beneficios de Seguridad
- **Credenciales Aisladas**: Cloudflare D1 no expone ningún token de AWS.
- **Resiliencia**: Aunque la cuenta de AWS sufra mantenimiento o caídas, la red Edge de Cloudflare continúa sirviendo las peticiones de los 100.000 usuarios sin interrupción.
