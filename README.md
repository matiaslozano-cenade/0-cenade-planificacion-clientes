# Panel de Planificación — Clientes

Panel de carta Gantt mensual para visualizar y crear eventos por cliente (reuniones, entregas, plazos, hitos críticos y revisiones). Datos persistidos en Supabase, deploy estático en Vercel.

## Stack

- **Frontend**: HTML + JS vanilla, sin build step. Tipografías DM Sans / DM Mono.
- **Backend**: Supabase (Postgres + RLS público) — proyecto `matiaslozano-cenade's Project carta gunt` (`wjbwccacjdkuejcriode`).
- **Hosting**: Vercel — team `matiaslozano-cenade's projects`.

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
events  (id, client_id → clients, type, start_date, end_date, description, created_at)
```

`events.type` está restringido a: `meeting`, `delivery`, `deadline`, `milestone`, `review`.

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
- Filtros por tipo de evento (chips activables)
- Stats del mes en curso (totales por tipo)
- Eventos single-day (puntos) y multi-día (barras)
- Modal "Nuevo cliente" para agregar clientes (nombre + rubro)
- Modal "Nuevo evento" que persiste en Supabase
- Eliminar cliente (botón × al hacer hover sobre la fila; borra eventos asociados en cascada)
- Exportar todo el dataset a CSV
