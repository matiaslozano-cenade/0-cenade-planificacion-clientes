# Panel de Planificación — Clientes

Panel de carta Gantt mensual para visualizar y crear eventos por cliente (reuniones, entregas, plazos, hitos críticos y revisiones). Datos persistidos en Supabase, deploy estático en Vercel.

## Stack

- **Frontend**: HTML + JS vanilla, sin build step. Tipografías Inter / JetBrains Mono.
- **Backend**: Supabase (Postgres + RLS público) — proyecto `matiaslozano-cenade's Project carta gunt` (`wjbwccacjdkuejcriode`).
- **Hosting**: Vercel — team `matiaslozano-cenade's projects`.

## Layout

- **Topbar** con marca, status pill (estado de conexión a Supabase) y acciones primarias.
- **Sidebar izquierda** (≥900 px) con stats del mes y filtros por tipo.
- **Main** con toolbar (mes, hoy, contador) y la grilla Gantt con encabezados sticky de día y de cliente.

En pantallas <900 px la sidebar se oculta automáticamente.

## Estructura

```
.
├── index.html   # App completa (UI + lógica + cliente Supabase)
├── README.md
└── .gitignore
```

## Esquema de base de datos

```sql
clients (id, name, sub, created_at)
events  (id, client_id → clients, type,
         start_date, end_date,
         start_time, end_time,
         recurrence_group_id,
         description, created_at)
```

`events.type` está restringido a: `meeting`, `delivery`, `deadline`, `milestone`, `review`.

`recurrence_group_id` (uuid, nullable) agrupa eventos creados a partir de una misma regla de repetición — los eventos individuales se generan al guardar y comparten ese id, lo que permite eliminar toda la serie con un solo `DELETE`.

RLS habilitado en ambas tablas con políticas públicas (read / insert / update / delete con `using (true)`). Cualquier usuario con la URL puede leer y escribir — apropiado para uso interno sin login. Si más adelante hay datos sensibles, restringir las policies a `auth.role() = 'authenticated'` y agregar Supabase Auth.

## Configuración

El cliente Supabase se inicializa desde `index.html` con la URL del proyecto y la **publishable key** (segura para exponer en el frontend, su único poder está delimitado por RLS):

```js
const SUPABASE_URL = 'https://wjbwccacjdkuejcriode.supabase.co';
const SUPABASE_KEY = 'sb_publishable_pcI9qqrsLJuPw1fO4v3qQw_CL7VKxnt';
```

## Desarrollo local

No requiere build. Servir el directorio con cualquier servidor estático:

```bash
python3 -m http.server 8000
# o
npx serve
```

Y abrir <http://localhost:8000>.

## Keep-alive de Supabase

GitHub Action en `.github/workflows/keep-alive.yml` se ejecuta a las 12:00 UTC los días 1, 6, 11, 16, 21, 26 y 31 de cada mes (gap máximo 5 días). Inserta un cliente con nombre `.` y `sub = keepalive` y lo elimina inmediatamente — basta para que Supabase no marque el proyecto como inactivo (free tier pausa a los 7 días).

Trigger manual: pestaña Actions del repo → "Keep Supabase awake" → Run workflow. O por CLI:

```bash
gh workflow run "Keep Supabase awake"
```

## Deploy

Conectado a Vercel — cada push a `main` despliega producción automáticamente.

```bash
# Deploy manual (preview)
vercel
# Deploy a producción
vercel --prod
```

## Funciones

- Vista Gantt mensual con navegación mes a mes
- Filtros por tipo de evento desde la sidebar
- Stats del mes en sidebar (cards con conteo por tipo + total destacado)
- Eventos single-day (puntos) y multi-día (barras)
- Modal "Nuevo cliente" / "Editar cliente" (nombre + rubro)
- Doble-click sobre fila de cliente para editar
- Eliminar cliente con botón × en hover (borra eventos en cascada)
- Modal "Nuevo evento" con hora opcional (inicio / fin) y recurrencia (diaria, semanal, quincenal, mensual) hasta una fecha tope
- Generación automática de hasta 200 ocurrencias por serie con `recurrence_group_id` compartido
- Doble-click sobre un evento para editarlo (cliente, tipo, fechas, horas, descripción)
- Doble-click sobre el header de un día (o celda vacía) abre el **detalle del día** con tres secciones: "Todo el día" (eventos sin hora), un **timeline 08:00–18:00** con eventos posicionados por hora y duración real (overlap por columnas) y "Fuera de horario" para eventos antes/después del bloque laboral. Si es hoy, el timeline muestra una línea de "Ahora". Click en cualquier bloque para editarlo, o "+ Nuevo evento" para crear con la fecha pre-cargada
- Click derecho sobre evento (barra o punto) para eliminarlo; si pertenece a una serie, ofrece eliminar solo este o toda la serie
- Avatar con iniciales y color por cliente (hash determinístico)
- Encabezados de día y columna de cliente sticky
- Empty state cuando no hay clientes
- Exportar todo el dataset a CSV

## Atajos de teclado

| Tecla | Acción |
|---|---|
| `T` | Saltar al mes actual |
| `N` | Crear evento |
| `⌘/Ctrl + ←` / `→` | Mes anterior / siguiente |
| `Esc` | Cerrar modal |
