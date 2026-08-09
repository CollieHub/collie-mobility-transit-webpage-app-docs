# 📡 Especificación de Endpoints REST API (Hono.js)

Servicio unificado de rutas REST servido por Hono.js sobre la red global de Cloudflare Workers.

---

## 1. Catálogo Público Unificado
- **Endpoint**: `GET /v1/catalog/public`
- **Descripción**: Devuelve la lista completa de líneas, ramales y paradas publicadas.
- **Formato de Respuesta**:
```json
{
  "lines": [
    {
      "id": "line-sit-zarate",
      "code": "SIT-ZARATE",
      "name": "Sistema Integral de Transporte Zárate",
      "color": "#1A73E8",
      "branches": [
        {
          "id": "branch-500-1",
          "code": "500-1",
          "name": "Ramal Centro - Barrio Cementerio"
        }
      ]
    }
  ]
}
```

---

## 2. Telemetría de Colectivos en Vivo (con Bounding Box)
- **Endpoint**: `GET /v1/fleet/state?bbox=west,south,east,north`
- **Parámetros Query**:
  - `bbox` (Opcional): Cuadrante visible del mapa (ej. `-59.1000,-34.2000,-58.9000,-34.0000`).
- **Descripción**: Consulta el snapshot en tiempo real desde Cloudflare KV/D1.

---

## 3. Horarios e Itinerarios
- **Endpoint**: `GET /v1/timetables?branch_id=...&day_type=habil`
- **Descripción**: Devuelve los horarios fijados para una línea o ramal específico.
