ServiGo
ServiGo es una plataforma para que personas emprendedoras ofrezcan sus servicios y los usuarios puedan buscarlos, revisar su portafolio (galería de imágenes) y elegir al que más confianza les genere. Este repositorio contiene el esqueleto del proyecto (MVP) usando Next.js + TypeScript + Tailwind CSS en frontend y Supabase (Postgres) como backend y almacenamiento.

Resumen del stack elegido (Opción A)
Frontend: Next.js + React + TypeScript
Estilos: Tailwind CSS
Backend / BBDD / Auth / Storage: Supabase (Postgres + Auth + Storage)
Mapas: Mapbox o Google Maps (inicialmente Mapbox recomendado)
Hosting: Vercel (deploy automático desde GitHub)
Testing: Jest + React Testing Library + Cypress (E2E) — opcional en primera iteración
Objetivos del MVP
Registro e inicio de sesión para usuarios y empleados (Supabase Auth).
Perfiles de empleados con descripción y portafolio de imágenes (Supabase Storage).
Búsqueda por servicio y filtrado por proximidad (geolocalización de oficina).
Visualización de perfil con galería de imágenes y reseñas.
Calificaciones/reviews básicas.
Requisitos previos (local)
Node.js 18+ (recomendado)
Git
Cuenta en Supabase (https://supabase.com) y proyecto creado
Cuenta en Vercel (para deploy)
Mapbox token (si usas Mapbox) o API key de Google Maps
Clonar repositorio:

git clone git@github.com:shokrpower115/ServiGo.git
cd ServiGo
Quick Start (sugerencia de comandos para el esqueleto)
Crear app Next.js con TypeScript:
npx create-next-app@latest . --ts
Instalar dependencias principales:
npm install @supabase/supabase-js tailwindcss postcss autoprefixer classnames
# o con yarn/pnpm
Inicializar Tailwind:
npx tailwindcss init -p
Configurar archivos (ver más abajo en "Estructura sugerida" y ".env.local").
Scripts recomendados (package.json):

dev: next dev
build: next build
start: next start
lint: next lint
test: jest (si configuras)
Variables de entorno (archivo .env.local)
Crea un archivo .env.local en la raíz con al menos:

NEXT_PUBLIC_SUPABASE_URL=https://xxxxx.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=public-anon-key
SUPABASE_SERVICE_ROLE_KEY=service-role-key (solo en servidor)
NEXT_PUBLIC_MAPBOX_TOKEN=pk.xxxxx (si usas Mapbox)
NEXT_PUBLIC_ permite el uso en cliente.
SUPABASE_SERVICE_ROLE_KEY NUNCA exponerlo en el cliente; usarlo solo en funciones del servidor.
Setup inicial en Supabase (pasos)
Crea un nuevo proyecto en Supabase.
En Authentication > Providers, habilita Email (y/o Google).
En Storage, crea un bucket llamado profiles (o employee-gallery) y configúralo como público o privado según prefieras.
En SQL editor, ejecutar las migraciones iniciales (ejemplo abajo).
Obtener la URL y ANON KEY desde Settings > API y ponerlas en .env.local.
SQL inicial (puedes pegarlo en el SQL editor de Supabase):

-- Habilitar PostGIS si no está habilitado
create extension if not exists postgis;

-- Tabla de perfiles (vinculada a auth.users)
create table if not exists employee_profiles (
  id uuid primary key default gen_random_uuid(),
  user_id uuid references auth.users(id) on delete cascade,
  title text,
  bio text,
  services text[], -- array de servicios (tags)
  office_location geography(Point, 4326), -- lat/lon
  address text,
  price_min numeric,
  price_max numeric,
  rating_avg numeric default 0,
  verified boolean default false,
  created_at timestamptz default now(),
  updated_at timestamptz default now()
);

-- Tabla de servicios (opcional, normalizada)
create table if not exists services (
  id uuid primary key default gen_random_uuid(),
  name text not null,
  category text,
  created_at timestamptz default now()
);

-- Galería de imágenes
create table if not exists images (
  id uuid primary key default gen_random_uuid(),
  employee_id uuid references employee_profiles(id) on delete cascade,
  url text not null,
  caption text,
  is_feature boolean default false,
  created_at timestamptz default now()
);

-- Reseñas
create table if not exists reviews (
  id uuid primary key default gen_random_uuid(),
  customer_id uuid references auth.users(id) on delete set null,
  employee_id uuid references employee_profiles(id) on delete cascade,
  rating int check (rating >= 1 and rating <= 5),
  comment text,
  created_at timestamptz default now()
);

-- Favoritos
create table if not exists favorites (
  id uuid primary key default gen_random_uuid(),
  customer_id uuid references auth.users(id) on delete cascade,
  employee_id uuid references employee_profiles(id) on delete cascade,
  created_at timestamptz default now(),
  unique (customer_id, employee_id)
);
Notas:

Supabase usa auth.users para la gestión de usuarios; relacionamos employee_profiles.user_id con esa tabla.
office_location usa geography(Point, 4326) para consultas por distancia (PostGIS).
Para consultas de proximidad en SQL:
select *, ST_Distance(office_location, ST_GeogFromText('SRID=4326;POINT(lon lat)')) as distance
from employee_profiles
where ST_DWithin(office_location, ST_GeogFromText('SRID=4326;POINT(lon lat)'), radius_in_meters)
order by distance;
Estructura de carpetas sugerida
/src
/pages (Next.js pages)
/app (si usas app router)
/components
/lib (supabase client, helpers)
/hooks
/styles (tailwind config)
/utils
/public (assets)
/tests (e2e / unit)
/scripts (migrations, seed)
Ejemplo de archivo importante:

src/lib/supabaseClient.ts — cliente de supabase
src/pages/api/upload (si necesitas endpoints para firma o procesamiento)
Tareas iniciales (README -> backlog / Issues)
A continuación un backlog inicial con prioridad para el MVP. Puedes crear issues en GitHub a partir de estas tareas.

Tareas críticas (MVP)

 1. Inicializar proyecto Next.js + TypeScript + Tailwind (esqueleto del repo)
 2. Configurar Supabase: crear proyecto, bucket profiles, keys y variables de entorno
 3. Ejecutar migración SQL en Supabase (tablas básicas)
 4. Integrar Supabase Auth en Next.js (registro/login/email)
 5. Crear formulario para completar perfil de empleado (incluye lat/lon o búsqueda de dirección)
 6. Implementar subida de imágenes a Supabase Storage y mostrar galería en perfil
 7. Implementar búsqueda básica por servicio (filtros) y listado con distancia
 8. Integrar mapa (Mapbox) para visualizar marcadores y/o buscar por proximidad
 9. Implementar creación y visualización de reseñas (reviews) y cálculo de rating_avg
 10. Pruebas básicas y documentación en README
 11. Configurar deployment automático a Vercel
Tareas de mejora (post-MVP)

 12. Implementar Algolia/MeiliSearch para búsquedas rápidas y relevancia
 13. Añadir roles y verificación de empleados (document upload / verificación)
 14. Pagos (Stripe) y modelos de monetización
 15. Notificaciones (email/SMS) y mensajería interna
 16. Tests E2E (Cypress)
 17. Monitorización (Sentry) y analítica
Criterios de aceptación (MVP)
Un usuario puede registrarse y loguearse.
Un empleado puede completar su perfil y subir al menos 3 imágenes.
Un usuario puede buscar por servicio y ver resultados ordenados por proximidad (o mostrar distancia).
Al visitar el perfil de un empleado se muestra la bio, servicios, imágenes y reseñas.
Las imágenes se sirven eficientemente desde Supabase Storage (idealmente con CDN).
Buenas prácticas y convenciones
Usa TypeScript estrictamente (strict mode).
Mantén componentes pequeños y reutilizables.
Usa RLS (Row Level Security) en Supabase si vas a permitir modificaciones en tablas por parte de usuarios.
Protege las rutas sensibles con verificación de roles.
Versiona migraciones SQL (scripts en /scripts/migrations).
Roadmap propuesto (3 meses)
Semana 1: Scaffolding, Supabase setup, auth y modelos de datos.
Semana 2: Formularios de perfil, upload de imágenes y galería.
Semana 3–4: Búsqueda por servicio y proximidad, integración de mapa.
Mes 2: Reseñas, favoritos y mejoras de UX.
Mes 3: Tests, revisión de seguridad, deploy y beta testing.
Próximo paso que puedo hacer por ti
Puedo generar ahora el scaffolding básico (archivos iniciales) para:

README (este)
.gitignore
package.json (sugerido)
src/lib/supabaseClient.ts
pages/_app.tsx y un ejemplo de página index y perfil
tailwind.config.js y postcss.config.js
ejemplo .env.local.example
¿Quieres que genere el esqueleto de archivos ahora en la conversación (te los muestro aquí para que los pegues en tu repo), o prefieres que te dé instrucciones para crearlo localmente paso a paso? Indícame qué prefieres y procedo.
