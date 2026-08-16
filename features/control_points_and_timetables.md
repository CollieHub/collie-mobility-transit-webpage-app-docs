# 📍 Puntos de Control y Grillas de Horarios

Documentación del sistema de gestión, procesamiento cartográfico y visualización de **Puntos de Control** e **Itinerarios de Transporte** en la plataforma **Collie Transit**.

---

## 🎯 ¿Qué es un Punto de Control?

Un **Punto de Control** (o *Waypoint de Horario*) es una parada física clave dentro del recorrido de un ramal a la cual se le asignan horarios específicos de paso o control de despachos en la grilla de horarios.

---

## 🗄️ 1. Persistencia y Desnormalización en Base de Datos D1

Los Puntos de Control se almacenan de dos formas sincronizadas para garantizar integridad y máxima velocidad de lectura:

1. **Definición Maestra en `schedules`**:
   - `headers_json`: Títulos de cabecera de las columnas.
   - `stop_addresses_json`: Lista ordenada de paradas asignadas.
   - `stop_mappings_json`: Mapa de índices a paradas físicas.

2. **Atributo Directo en `stops.is_control_point` ($O(1)$)**:
   - La tabla `stops` cuenta con la columna `is_control_point INTEGER DEFAULT 0`.
   - **`1`**: La parada actúa como Punto de Control en ese ramal y sentido.
   - **`0`**: La parada es una parada regular intermedia.
   - **Sincronización Automática en CRUD**: Al guardar cualquier grilla de horarios en `/v1/admin/schedules/batch`, el backend actualiza de forma atómica la columna `is_control_point` en la tabla `stops` para ese ramal y sentido.

---

## 🧠 2. Identificación en el Mapa y Aplicaciones Móviles

El visor cartográfico (`TransitMap.tsx`) y las aplicaciones de Kotlin / Swift evalúan directamente:
```typescript
const isControlPoint = stop.is_control_point === 1;
```
Esto garantiza **cero latencia de parseo ($O(1)$)**, determinismo total y la completa eliminación de falsos positivos por nombres similares.

---

## 🎨 3. Experiencia Visual y Modos de Usuario

| Elemento | Usuario Público (Sin Loguear) | Administrador / Inspección |
| :--- | :--- | :--- |
| **Color del Icono** | **Dorado / Ámbar (`#f59e0b`)** | **Dorado / Ámbar (`#f59e0b`)** con sombra paralela |
| **Forma del Icono** | Parada de colectivo estándar (`createStopIcon`) | Icono de reloj con aguja interna (`createWaypointCircleDotIcon`) |
| **Tamaño Visual** | **1.0x** (mismo tamaño que paradas normales) | **1.4x** (ligeramente ampliado) |
| **Botón de Control** | Botón de Reloj **`🕒`** en cabecera | Botón de Reloj **`🕒`** + Secuenciador **`#`** + Vinculador **`🔗`** |
| **Etiqueta Flotante (`<Tooltip>`)** | Controlada interactivamente con el botón **`🕒`** | Controlada interactivamente con el botón **`🕒`** |
| **Contenido de la Etiqueta** | Horas programadas de unidades activas (`#5: 14:10`) o nombre del punto de control | Horas programadas de unidades activas (`#5: 14:10`) o nombre del punto de control |

---

## ⏱️ 4. Horarios en Tiempo Real de Unidades Activas

Cuando un colectivo activo circula por el ramal:
1. El mapa extrae la lista de horarios programados (`bus.stopTimes`) correspondientes al despacho del vehículo.
2. Cada Punto de Control calcula las horas de paso de los colectivos en tránsito para esa dirección.
3. El cartel flotante superior despliega automáticamente las horas formateadas (ej. `RZ01 #5: 13:40`, `RZ01 #5: 14:08`), permitiendo a los pasajeros saber exactamente a qué hora pasará cada coche.
