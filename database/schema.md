# 🗄️ Esquema de Base de Datos Cloudflare D1 (SQLite)

Este esquema define la estructura de datos SQL optimizada que almacena la información **publicada** lista para consumo en Cloudflare D1.

---

## 📜 Definición DDL de Tablas (SQLite / Cloudflare D1)

```sql
-- 1. Líneas de Colectivo Publicadas
CREATE TABLE IF NOT EXISTS lines (
    id TEXT PRIMARY KEY,
    code TEXT NOT NULL UNIQUE,
    name TEXT NOT NULL,
    color TEXT DEFAULT '#000000',
    jurisdiction TEXT DEFAULT 'Municipal',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- 2. Ramales de cada Línea
CREATE TABLE IF NOT EXISTS branches (
    id TEXT PRIMARY KEY,
    line_id TEXT NOT NULL,
    code TEXT NOT NULL,
    name TEXT NOT NULL,
    description TEXT,
    FOREIGN KEY (line_id) REFERENCES lines(id) ON DELETE CASCADE
);

-- 3. Paradas de Colectivos
CREATE TABLE IF NOT EXISTS stops (
    id TEXT PRIMARY KEY,
    branch_id TEXT NOT NULL,
    direction TEXT CHECK(direction IN ('ida', 'vuelta')),
    stop_order INTEGER NOT NULL,
    name TEXT NOT NULL,
    lat REAL NOT NULL,
    lng REAL NOT NULL,
    proj_lat REAL,
    proj_lng REAL,
    FOREIGN KEY (branch_id) REFERENCES branches(id) ON DELETE CASCADE
);

-- 4. Trazado Vectorial de Recorridos (Shapes)
CREATE TABLE IF NOT EXISTS route_shapes (
    id TEXT PRIMARY KEY,
    branch_id TEXT NOT NULL,
    direction TEXT CHECK(direction IN ('ida', 'vuelta')),
    coordinates_json TEXT NOT NULL, -- Array JSON: [[lat, lng], [lat, lng], ...]
    total_distance_km REAL DEFAULT 0,
    FOREIGN KEY (branch_id) REFERENCES branches(id) ON DELETE CASCADE
);

-- 5. Horarios de Salida (Timetables)
CREATE TABLE IF NOT EXISTS timetables (
    id TEXT PRIMARY KEY,
    branch_id TEXT NOT NULL,
    direction TEXT CHECK(direction IN ('ida', 'vuelta')),
    day_type TEXT CHECK(day_type IN ('habil', 'sabado', 'domingo', 'feriado')),
    departure_time TEXT NOT NULL, -- HH:MM (ej. 06:15)
    dispatch_order INTEGER,
    FOREIGN KEY (branch_id) REFERENCES branches(id) ON DELETE CASCADE
);

-- Índices de consulta ultra-rápida
CREATE INDEX IF NOT EXISTS idx_stops_branch ON stops(branch_id, direction);
CREATE INDEX IF NOT EXISTS idx_timetables_branch ON timetables(branch_id, day_type);
```
