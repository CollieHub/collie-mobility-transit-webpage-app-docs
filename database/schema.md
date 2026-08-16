# 🗄️ Esquema Relacional de Base de Datos (Cloudflare D1)

Este documento especifica el modelo relacional de datos de **Cloudflare D1** (`collie-mobility-transit-db`) para la aplicación Full-Stack Edge **`collie-mobility-transit-webpage-app`**. Todas las claves primarias (`id`) y claves foráneas en las 9 tablas utilizan exclusivamente el formato **UUID v4 / UUID v5**.

---

## 📐 Diagrama Entidad-Relación (Mermaid)

```mermaid
erDiagram
    companies ||--o{ lines : "opera"
    companies ||--o{ branches : "gestiona"
    lines ||--o{ branches : "contiene"
    branch_statuses ||--o{ branches : "define_estado"
    branches ||--o{ stops : "posee"
    branches ||--o{ route_shapes : "dibuja"
    branches ||--o{ schedules : "programa"
    day_types ||--o{ schedules : "clasifica"
    schedules ||--o{ schedule_items : "contiene_despachos"

    companies {
        TEXT id PK "UUID e.g. 23ccf89e-8e47-5a03-836c-4a3998975f6a"
        TEXT code UK "e.g. SIT"
        TEXT name "Nombre de la empresa prestadora"
        TEXT description "Descripción de la cobertura"
        TIMESTAMP created_at "Fecha de creación"
    }

    lines {
        TEXT id PK "UUID e.g. 05df921c-b6ad-59fb-8e08-ca0adab197ca"
        TEXT code UK "e.g. RZ01"
        TEXT name "Nombre descriptivo de la línea"
        TEXT color "Código Hexadecimal de color"
        TEXT jurisdiction "Municipal / Provincial"
        TEXT company_id FK "UUID Relación con empresas.id"
        TEXT company "Nombre textual de la empresa"
        TIMESTAMP created_at "Fecha de alta"
    }

    branch_statuses {
        TEXT id PK "UUID e.g. f82c4109-ba8e-521a-a093-07db0825cf3a"
        TEXT code UK "e.g. active"
        TEXT name "Nombre visual del estado (e.g. Activo)"
        TEXT description "Descripción operativa del estado"
        TEXT color "Código Hexadecimal de estado"
        TIMESTAMP created_at "Fecha de creación"
    }

    branches {
        TEXT id PK "UUID e.g. 644f8b84-ab93-5d46-a74e-bfd979acd1ea"
        TEXT line_id FK "UUID Relación con lines.id"
        TEXT code "Código de ramal (e.g. RZ01)"
        TEXT name "Nombre del recorrido o variante"
        TEXT company_id FK "UUID Relación con empresas.id"
        TEXT company "Nombre textual de la empresa"
        TEXT branch_statuses_id FK "UUID Relación con branch_statuses.id"
        TEXT description "Descripción del ramal"
    }

    stops {
        TEXT id PK "UUID e.g. 4a9f3b84-ab93-5d46-a74e-bfd979acd1ea"
        TEXT branch_id FK "UUID Relación con branches.id"
        TEXT direction "Sentido: ida / vuelta"
        INTEGER stop_order "Secuencia ordinal de parada"
        TEXT name "Dirección física de la parada"
        REAL lat "Latitud WGS84"
        REAL lng "Longitud WGS84"
        REAL proj_lat "Latitud proyectada en calle"
        REAL proj_lng "Longitud proyectada en calle"
        INTEGER is_control_point "1 = Punto de Control / 0 = Parada Regular"
    }

    route_shapes {
        TEXT id PK "UUID e.g. 7b9f3b84-ab93-5d46-a74e-bfd979acd1ea"
        TEXT branch_id FK "UUID Relación con branches.id"
        TEXT direction "Sentido: ida / vuelta"
        TEXT coordinates_json "JSON Array [[lat, lng], ...]"
        REAL total_distance_km "Distancia total del trazado en km"
    }

    day_types {
        TEXT id PK "UUID e.g. 88f18fc3-ba8e-521a-a093-07db0825cf3a"
        TEXT code UK "e.g. lunes_a_viernes"
        TEXT name "Nombre visual del tipo de día"
        TEXT description "Descripción del tipo de servicio"
        INTEGER display_order "Orden visual en el combo desplegable"
        TEXT aws_schedule_type_prefix "Mapeo con AWS DynamoDB"
        INTEGER is_enabled "1 = Habilitado / 0 = Deshabilitado"
    }

    schedules {
        TEXT id PK "UUID e.g. 5bca91e8-2c91-5c3c-9111-00669ac8d4db"
        TEXT branch_id FK "UUID Relación con branches.id"
        TEXT direction "Sentido: ida / vuelta"
        TEXT day_types_id FK "UUID Relación con day_types.id"
        TEXT name "Nombre descriptivo de la grilla"
        TEXT headers_json "JSON Array de cabeceras resueltas"
        TEXT header_aliases_json "JSON Array de alias cargados en consola"
        TEXT stop_addresses_json "JSON Array de direcciones físicas de paradas"
        TIMESTAMP created_at "Fecha de creación"
    }

    schedule_items {
        TEXT id PK "UUID e.g. 0027844c-9a8b-57cf-aa21-74cbd6e85ee8"
        TEXT schedule_id FK "UUID Relación con schedules.id"
        TEXT departure_time "Hora de salida inicial (HH:MM)"
        INTEGER dispatch_order "Orden en la planilla del día"
        TEXT trip_times_json "JSON Array de horarios por punto"
        TIMESTAMP created_at "Fecha de creación"
    }
```

---

## 📊 Descripción de las 9 Tablas Relacionales (Todas con Identificador UUID)

### 1. `companies` (Empresas de Colectivos)
- Contiene los datos institucionales de las empresas de transporte (`SIT`, `228 (San Isidro)`).
- **Clave Primaria**: `id` (UUID v4).

### 2. `lines` (Líneas de Colectivos)
- Almacena cada línea comercial o tarifa.
- **Clave Primaria**: `id` (UUID v4).
- **Clave Foránea**: `company_id` (UUID) -> `companies(id)`.

### 3. `branch_statuses` (Estados Operativos de Ramal)
- Define el estado operativo del servicio (`active`, `interrupted`, `reduced`, `suspended`).
- **Clave Primaria**: `id` (UUID v4).

### 4. `branches` (Ramales / Recorridos)
- Almacena los ramales y variantes de cada línea.
- **Clave Primaria**: `id` (UUID v4).
- **Claves Foráneas**: `line_id` (UUID) -> `lines(id)`, `company_id` (UUID) -> `companies(id)`, `branch_statuses_id` (UUID) -> `branch_statuses(id)`.

### 5. `stop_groups` (Grupos de Paradas Unificadas / Estaciones)
- Agrupa múltiples paradas físicas de distintos ramales que comparten un nodo de espera (ej. "Terminal NK", "Plaza Italia").
- **Clave Primaria**: `id` (UUID v4).
- **Campos**: `code` (UK), `name`, `lat`, `lng`, `description`, `is_enabled`.

### 5b. `stop_group_details` (Detalles y Coordenadas Específicas del Grupo)
- Almacena sub-ubicaciones, coordenadas exactas de refugios, dársenas o plataformas dentro del grupo de paradas.
- **Clave Primaria**: `id` (UUID v4).
- **Clave Foránea**: `stop_group_id` (UUID) -> `stop_groups(id)`.
- **Campos**: `name`, `lat`, `lng`, `address`, `platform_code`, `description`, `display_order`.

### 6. `stops` (Paradas Físicas por Ramal)
- Paradas geolocalizadas por ramal y sentido (`ida` / `vuelta`).
- **Clave Primaria**: `id` (UUID v4).
- **Claves Foráneas**: `branch_id` (UUID) -> `branches(id)`, `stop_group_id` (UUID) -> `stop_groups(id)`.

### 6. `route_shapes` (Trazados Vectoriales)
- Coordenadas geográficas para renderizar el recorrido en los mapas interactivos.
- **Clave Primaria**: `id` (UUID v4).
- **Clave Foránea**: `branch_id` (UUID) -> `branches(id)`.

### 7. `day_types` (Tipos de Día)
- Definición de tipos de servicio o combos con identificador único UUID (`id`).
- **Campos Destacados**: `is_enabled` (INTEGER: `1` = Habilitado / `0` = Deshabilitado). Permite ocultar dinámicamente horarios estacionales o extraordinarios (e.g. Horario de Invierno `especial`).
- **Clave Primaria**: `id` (UUID v4).

### 8. `schedules` (Grilla de Horarios Maestro)
- Cabecera de la grilla de horarios por ramal, sentido y tipo de día. Almacena las paradas de control y alias.
- **Clave Primaria**: `id` (UUID v4).
- **Claves Foráneas**: `branch_id` (UUID) -> `branches(id)`, `day_types_id` (UUID) -> `day_types(id)`.

### 9. `schedule_items` (Despachos e Horarios Individuales)
- Filas de servicios/despachos individuales que componen la grilla de horarios.
- **Clave Primaria**: `id` (UUID v4).
- **Clave Foránea**: `schedule_id` (UUID) -> `schedules(id)`.
