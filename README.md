# 📚 Collie Mobility Transit Webpage App - Documentación Oficial

Bienvenido a la documentación oficial del proyecto **`collie-mobility-transit-webpage-app`**.

---

## 🎯 Objetivo de la Arquitectura

Este proyecto implementa un **Escudo de Aislamiento Financiero y de Rendimiento (Public Edge Shield)**:
- **Cero Exposición de AWS**: La consola e infraestructura de AWS se mantiene exclusivamente como **Backoffice Interno** reservado para la administración y carga de datos.
- **Servidor Público en Cloudflare**: Los clientes finales (**Webpage App, Android, iOS y Radar**) consumen única y exclusivamente la red global Edge de Cloudflare (**$0.00 USD Egress Fee**).
- **Base de Datos Pública Distribuida**: La información publicada (Líneas, Ramales, Paradas, Horarios y Recorridos) se sirve desde **Cloudflare D1 (SQL)** y **Cloudflare KV** a latencia ultra-baja (<10 ms).

---

## 📂 Contenido de la Documentación

1. 🏗️ [Visión General de Arquitectura](file:///Users/jonatandanielmoreira/developer/proyectos/collie-mobility-towing/collie-mobility-transit-webpage-app-docs/architecture/overview.md)
2. 🗄️ [Esquema de Base de Datos Cloudflare D1 (SQL)](file:///Users/jonatandanielmoreira/developer/proyectos/collie-mobility-towing/collie-mobility-transit-webpage-app-docs/database/schema.md)
3. 📡 [Especificación de Endpoints REST API (Hono.js)](file:///Users/jonatandanielmoreira/developer/proyectos/collie-mobility-towing/collie-mobility-transit-webpage-app-docs/api/endpoints.md)
4. 🔄 [Pipeline de Sincronización (AWS Backoffice ➔ Cloudflare D1)](file:///Users/jonatandanielmoreira/developer/proyectos/collie-mobility-towing/collie-mobility-transit-webpage-app-docs/sync/aws_to_cloudflare.md)

---

## 🛠️ Tecnologías Utilizadas

- **Frontend Web**: React 19 + TypeScript + Leaflet Maps
- **Backend API Server**: Hono.js sobre Cloudflare Workers
- **Database Engine**: Cloudflare D1 (SQLite) + Cloudflare KV Storage
- **Deployment**: Cloudflare Pages & Workers
