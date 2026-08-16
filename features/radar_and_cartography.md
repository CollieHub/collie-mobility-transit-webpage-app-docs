# 📡 Editor Cartográfico Radar & Trazados

Documentación del editor de recorridos, manipulación de paradas y algoritmos de proyección geométrica en **Collie Transit** (`RadarView.tsx`).

---

## 🧭 1. Reordenamiento Secuencial de Paradas (1..N)

El botón **"Reordenar por Avance del Recorrido"** (`Compass`) permite ordenar automáticamente la totalidad de las paradas de un ramal según la distancia acumulada sobre el trazado desde la cabecera inicial hasta la terminal de destino.

### Algoritmo de Distancia Acumulada sobre Polyline
1. Para cada parada física, calcula su proyección ortogonal más cercana sobre cada segmento de la polyline del recorrido.
2. Suma la distancia geodésica acumulada de los segmentos anteriores más la distancia fraccional sobre el segmento contenedor.
3. Ordena ascendentemente todas las paradas asignando `stop_order: 1..N`.

---

## 📐 2. Algoritmo de Desplazamiento Perpendicular a 6 Metros (`offsetPointToRightOfPolyline`)

Para asegurar que las paradas no se ubiquen sobre el centro de la calle (eje del trazado) ni en la mano contraria de circulación, el sistema ejecuta un desplazamiento perpendicular automático de **6 metros hacia la derecha del vector de avance**:

```typescript
function offsetPointToRightOfPolyline(
  pt: [number, number],
  polyline: [number, number][],
  offsetMeters: number = 6
): [number, number] {
  if (!polyline || polyline.length < 2) return pt;

  // 1. Encuentra el segmento más cercano y su proyección
  let segIdx = 0;
  let projPt: [number, number] = pt;
  // ... cálculo de proyección sobre polyline ...

  const a = polyline[segIdx];
  const b = polyline[segIdx + 1];

  const midLat = (a[0] + b[0]) / 2;
  const radLat = (midLat * Math.PI) / 180;
  const cosLat = Math.cos(radLat);

  // 2. Componentes vectoriales de dirección en espacio equivalente a metros
  const dx = (b[1] - a[1]) * cosLat;
  const dy = b[0] - a[0];
  const len = Math.sqrt(dx * dx + dy * dy);

  if (len === 0) return projPt;

  // Vector unitario de avance (ux, uy)
  const ux = dx / len;
  const uy = dy / len;

  // 3. Vector perpendicular horario (hacia la derecha): (uy, -ux)
  const nx = uy;
  const ny = -ux;

  // 4. Conversión de offset en metros a grados geográficos
  const deltaDeg = offsetMeters / 111320;
  const rightLat = projPt[0] + ny * deltaDeg;
  const rightLng = projPt[1] + (nx * deltaDeg) / cosLat;

  return [rightLat, rightLng];
}
```

---

## 🛠️ 3. Herramientas de la Barra de Paradas

- **Proyectar Paradas (`MapPin`)**: Calcula la proyección perpendicular de todas las paradas sobre la calle más cercana.
- **Invertir Secuencia (`ArrowUpDown`)**: Invierte el orden `1..N` a `N..1` al crear ramales de vuelta.
- **Reordenar y Acomodar a 6m (`Compass`)**: Secuencia todas las paradas y las desplaza a 6m a la derecha de la calzada.
- **Replicar Paradas (`Copy`)**: Clona el set completo de paradas hacia otro ramal hermano.
- **Autogenerar por Distancia (`Sparkles`)**: Crea paradas intermedias automáticamente cada X metros (ej. cada 300m o 400m).
