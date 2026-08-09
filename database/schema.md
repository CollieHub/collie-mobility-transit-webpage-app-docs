# 🗄️ Esquema Relacional de Base de Datos (Cloudflare D1)

Este documento especifica el modelo relacional de datos de **Cloudflare D1** (`collie-mobility-transit-db`) para la aplicación Full-Stack Edge **`collie-mobility-transit-webpage-app`**.

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

    companies {
        TEXT id PK "e.g. company-sit"
        TEXT code UK "e.g. SIT"
        TEXT name "Nombre de la empresa prestadora"
        TEXT description "Descripción de la cobertura"
        TIMESTAMP created_at "Fecha de creación"
    }

    lines {
        TEXT id PK "e.g. line-RZ01"
        TEXT code UK "e.g. RZ01"
        TEXT name "Nombre descriptivo de la línea"
        TEXT color "Código Hexadecimal de color"
        TEXT jurisdiction "Municipal / Provincial"
        TEXT company_id FK "Relación con empresas.id"
        TEXT company "Nombre textual de la empresa"
        TIMESTAMP created_at "Fecha de alta"
    }

    branch_statuses {
        TEXT id PK "e.g. status-active"
        TEXT code UK "e.g. active"
        TEXT name "Nombre visual del estado (e.g. Activo)"
        TEXT description "Descripción operativa del estado"
        TEXT color "Código Hexadecimal de estado"
        TIMESTAMP created_at "Fecha de creación"
    }

    branches {
        TEXT id PK "e.g. route-30877"
        TEXT line_id FK "Relación con lines.id"
        TEXT code "Código de ramal (e.g. RZ01)"
        TEXT name "Nombre del recorrido o variante"
        TEXT company_id FK "Relación con empresas.id"
        TEXT company "Nombre textual de la empresa"
        TEXT branch_statuses_id FK "Relación con branch_statuses.id"
        TEXT description "Descripción del ramal"
    }

    stops {
        TEXT id PK "e.g. stop-route-30877-ida-1"
        TEXT branch_id FK "Relación con branches.id"
        TEXT direction "Sentido: ida / vuelta"
        INTEGER stop_order "Secuencia ordinal de parada"
        TEXT name "Dirección física de la parada"
        REAL lat "Latitud WGS84"
        REAL lng "Longitud WGS84"
        REAL proj_lat "Latitud proyectada en calle"
        REAL proj_lng "Longitud proyectada en calle"
    }

    route_shapes {
        TEXT id PK "e.g. shape-route-30877-ida"
        TEXT branch_id FK "Relación con branches.id"
        TEXT direction "Sentido: ida / vuelta"
        TEXT coordinates_json "JSON Array [[lat, lng], ...]"
        REAL total_distance_km "Distancia total del trazado en km"
    }

    day_types {
        TEXT id PK "e.g. lunes_a_viernes"
        TEXT code UK "e.g. lunes_a_viernes"
        TEXT name "Nombre visual del tipo de día"
        TEXT description "Descripción del tipo de servicio"
        INTEGER display_order "Orden visual en el combo desplegable"
        TEXT aws_schedule_type_prefix "Mapeo con AWS DynamoDB"
    }

    schedules {
        TEXT id PK "e.g. sch-route-30877-ida-lunes_a_viernes-1"
        TEXT branch_id FK "Relación con branches.id"
        TEXT direction "Sentido: ida / vuelta"
        TEXT day_types_id FK "Relación con day_types.id"
        TEXT departure_time "Hora de salida inicial (HH:MM)"
        INTEGER dispatch_order "Orden en la planilla del día"
        TEXT trip_times_json "JSON Array de horarios por punto"
        TEXT headers_json "JSON Array de cabeceras resueltas"
        TEXT header_aliases_json "JSON Array de alias cargados en consola"
        TEXT stop_addresses_json "JSON Array de direcciones físicas de paradas"
    }
```

---

## 📊 Descripción de las 8 Tablas Relacionales (Todas en Plural)

### 1. `companies` (Empresas de Colectivos)
- Contiene los datos institucionales de las empresas de transporte (`SIT`, `228 (San Isidro)`).
- **Clave Primaria**: `id` (Ej: `company-sit`, `company-228`).

### 2. `lines` (Líneas de Colectivos)
- Almacena cada línea comercial o tarifa.
- **Clave Foránea**: `company_id` -> `companies(id)`.

### 3. `branch_statuses` (Estados Operativos de Ramal)
- Define el estado operativo del servicio (`active`, `interrupted`, `reduced`, `suspended`).
- **Clave Primaria**: `id` (Ej: `status-active`, `status-interrupted`).

### 4. `branches` (Ramales / Recorridos)
- Almacena los ramales y variantes de cada línea.
- **Claves Foráneas**: `line_id` -> `lines(id)`, `company_id` -> `companies(id)`, `branch_statuses_id` -> `branch_statuses(id)`.

### 5. `stops` (Paradas Físicas)
- Paradas geolocalizadas por ramal y sentido (`ida` / `vuelta`).
- **Clave Foránea**: `branch_id` -> `branches(id)`.

### 6. `route_shapes` (Trazados Vectoriales)
- Coordenadas geográficas para renderizar el recorrido en los mapas interactivos.
- **Clave Foránea**: `branch_id` -> `branches(id)`.

### 7. `day_types` (Tipos de Día)
- Definición de tipos de servicio o combos para filtrado de horarios con secuencia visual (`display_order`):
  1. `Lunes a Viernes`
  2. `Sábados`
  3. `Domingos y Feriados`
  4. `Especial (Horario Extraordinario / Invierno)`

### 8. `schedules` (Horarios y Puntos Intermedios)
- Planilla con horarios de salida y matriz de tiempos de paso por paradas de control.
- **Claves Foráneas**: `branch_id` -> `branches(id)`, `day_types_id` -> `day_types(id)`.
