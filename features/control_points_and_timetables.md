# 📍 Puntos de Control y Grillas de Horarios

Documentación del sistema de gestión, procesamiento cartográfico y visualización de **Puntos de Control** e **Itinerarios de Transporte** en la plataforma **Collie Transit**.

---

## 🎯 ¿Qué es un Punto de Control?

Un **Punto de Control** (o *Waypoint de Horario*) es una parada física clave dentro del recorrido de un ramal a la cual se le asignan horarios específicos de paso o control de despachos en la grilla de horarios.

---

## 🖥️ 1. Configuración en la Consola de Administración (`/login?tab=schedules`)

En el panel de administración (**Consola Horarios**), los operadores configuran las columnas de horarios para cada ramal y tipo de día (Lunes a Viernes, Sábado, Domingo, Feriados, Invierno):

1. **Encabezados de Control (`headers`)**:
   - Nombres de referencia del punto de control (ej. `ESCALADA`, `HOSPITAL`, `PLAZA ITALIA`, `TERMINAL FONAVI`).
2. **Mapeo de Paradas Físicas (`stopAddresses` / `stopMappings`)**:
   - Debajo de cada encabezado, un desplegable interactivo permite seleccionar la parada física exacta del ramal (ej. `1. RVVP+45 Escalada`, `31. Dr. Félix Pagola 380`).
3. **Persistencia en Base de Datos Cloudflare D1**:
   - Al guardar la grilla, los datos se almacenan en la tabla `schedules` en campos JSON estructurados:
     - `headers_json`: Lista de títulos de columnas.
     - `stop_addresses_json`: Arreglo ordenado con los nombres completos de las paradas asociadas.
     - `stop_mappings_json`: Diccionario clave-valor que relaciona el índice de la columna con la parada asignada.
     - `matrix_json`: Matriz de horas programadas para cada servicio/despacho.

---

## 🧠 2. Algoritmo de Identificación en el Mapa (`matchesDeclaredControlStop`)

Para evitar falsos positivos por coincidencias parciales de texto (por ejemplo, evitar que la palabra `"Hospital"` marque paradas secundarias de la misma calle), se utiliza la función estricta `matchesDeclaredControlStop` en `TransitMap.tsx`:

```typescript
function matchesDeclaredControlStop(declaredStr: string, stopId: string, stopName: string): boolean {
  if (!declaredStr) return false;
  
  // 1. Coincidencia exacta de ID
  if (stopId && declaredStr === stopId) return true;

  const normDeclared = normalizeStopName(declaredStr);
  const cleanDeclared = normalizeStopName(declaredStr.replace(/^\d+[\.\s\-]+\s*/, ''));

  const normStop = normalizeStopName(stopName);
  const cleanStop = normalizeStopName((stopName || '').replace(/^\d+[\.\s\-]+\s*/, ''));

  if (!normDeclared || !normStop) return false;

  // 2. Coincidencia exacta de texto normalizado o limpio sin prefijo numérico
  if (normDeclared === normStop || cleanDeclared === cleanStop || normDeclared === cleanStop || cleanDeclared === normStop) {
    return true;
  }

  // 3. Coincidencia por prefijo/subcadena para encabezados concisos (ej. "Estación", "Barrio España")
  if (normDeclared.length >= 4 && normStop.length >= 4) {
    if (normStop.startsWith(normDeclared) || normDeclared.startsWith(normStop) || cleanStop.startsWith(cleanDeclared) || cleanDeclared.startsWith(cleanStop)) {
      return true;
    }
  }

  return false;
}
```

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
