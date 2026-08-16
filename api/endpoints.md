# 📡 Especificación de Endpoints REST API (Hono.js)

Servicio unificado de rutas REST servido por **Hono.js** sobre la red global Edge de **Cloudflare Workers**.

---

## 1. Catálogo Público Unificado
- **Endpoint**: `GET /v1/catalog/public`
- **Descripción**: Devuelve la lista completa de empresas, líneas y ramales activos.

---

## 2. Detalle Cartográfico y Trazados por Ramal
- **Endpoint**: `GET /v1/catalog/public/data?ids=branch-id-1,branch-id-2`
- **Descripción**: Devuelve coordenadas geográficas detalladas de paradas, sentidos de ida/vuelta, waypoints y shapes de las polylines.

---

## 3. Horarios e Itinerarios de Ramal (D1 Timetables)
- **Endpoint**: `GET /v1/catalog/public/timetables?route_id=RZ01`
- **Query Params**:
  - `route_id`: Código o UUID del ramal (ej. `RZ01`, `RZ02`, `RZ03`).
  - `source` (Opcional): `ActiveSchedules` para forzar la grilla activa de producción.
- **Formato de Respuesta**:
```json
{
  "success": true,
  "data": [
    {
      "direction": "ida",
      "headsign": "Terminal NK",
      "schedules": {
        "lunes_a_viernes_ida": {
          "dayType": "lunes_a_viernes",
          "direction": "ida",
          "headers": ["Bº BURGAR PINTO Y CALLE 6", "ESTACIÓN", "HOSPITAL", "AVELLANEDA Y RIVADAVIA", "PACHECO Y JUSTA LIMA", "TERMINAL LAVALLE 2400"],
          "stopAddresses": ["1. Ameghino 3216", "5. Av. Anta 400", "15. Dr. Félix Pagola 1400", "22. Av. Rivadavia 899", "32. Justa Lima de Atanasio 800", "40. Rio Colorado 98"],
          "stopMappings": {
            "0": "1. Ameghino 3216",
            "1": "5. Av. Anta 400",
            "2": "15. Dr. Félix Pagola 1400",
            "3": "22. Av. Rivadavia 899",
            "4": "32. Justa Lima de Atanasio 800",
            "5": "40. Rio Colorado 98"
          },
          "matrix": [
            ["05:00", "05:12", "05:20", "05:27", "05:35", "05:45"],
            ["05:30", "05:42", "05:50", "05:57", "06:05", "06:15"]
          ]
        }
      }
    }
  ]
}
```

---

## 4. Telemetría de Colectivos en Tiempo Real (con Bounding Box)
- **Endpoint**: `GET /v1/fleet/state?bbox=west,south,east,north`
- **Descripción**: Consulta el snapshot en tiempo real de unidades activas desde Cloudflare KV/D1 con interpolación de horarios de paso (`stopTimes`).

---

## 5. Endpoints de Administración de Base de Datos D1
- **Endpoints**:
  - `GET /v1/admin/table/schedules?limit=5000`
  - `POST /v1/admin/table/schedules`
  - `GET /v1/admin/table/stops?limit=10000`
  - `POST /v1/admin/table/stops`
  - `GET /v1/admin/table/ads`
  - `POST /v1/admin/table/ads`
- **Descripción**: Operaciones CRUD autenticadas con límites ampliados para edición masiva de paradas, horarios y anuncios publicitarios.

---

## 6. Anuncios Comerciales y Campañas de Afiliados (Mercado Libre)
- **Endpoint**: `GET /v1/transit/ads`
- **Descripción**: Devuelve la lista de campañas y banners activos desde Cloudflare D1 / KV con enlaces de redirección a Mercado Libre y comercios locales.
- **Formato de Respuesta**:
```json
{
  "success": true,
  "ads": [
    {
      "id": "meli-ofertas-01",
      "title": "Mercado Libre",
      "subtitle": "⚡ Ofertas del Día hasta 40% OFF y Envíos Rápidos",
      "redirectUrl": "https://www.mercadolibre.com.ar/ofertas",
      "color": "#FFE600",
      "border": "#E6CF00",
      "text": "#2D3277",
      "order": 1
    }
  ]
}
```

