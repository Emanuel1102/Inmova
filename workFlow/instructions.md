# Proyecto Integrador SaaS Multitenant — Inicio del desarrollo

Actúa como mi mentor técnico y ayúdame a desarrollar paso a paso mi Proyecto Integrador SaaS Multitenant. Quiero entender lo que hago y poder explicar el código durante la sustentación; no quiero recibir todo el proyecto de golpe.

## 1\. Contexto y objetivo

Debo construir individualmente una aplicación SaaS multitenant como proyecto académico, con una semana de desarrollo y aproximadamente 2–4 horas disponibles al día.

El nicho elegido es **gestión inmobiliaria**. Cada inmobiliaria podrá administrar:

- Propiedades.
- Interesados o leads.
- Visitas a propiedades.

El enunciado exige multitenancy mediante **un schema PostgreSQL por tenant**, autenticación y protección de rutas, CRUD, idempotencia, rate limiting, Docker, CI/CD y documentación.

## 2\. Tecnologías y decisiones confirmadas

- Next.js con App Router.
- TypeScript.
- Supabase como proveedor de PostgreSQL. Ya he creado el proyecto en Supabase.
- TypeORM como ORM; no utilizar Prisma ni añadir Knex de entrada.
- Zod para validar datos en tiempo de ejecución.
- JWT en cookie `httpOnly`; no almacenar el token en `localStorage`.
- Hash seguro de contraseñas.
- GitHub Actions para CI.
- Docker y Docker Compose al final del desarrollo, no al principio.

Antes de implementar todo el sistema, verifica que TypeORM puede conectarse correctamente a Supabase, crear un schema de prueba y consultar tablas dentro de un schema dinámico. Si existe una limitación técnica, explícala y propón la solución más sencilla que cumpla el enunciado.

## 3\. Arquitectura acordada

Utilizaremos un **monolito modular organizado por funcionalidades, con separación de responsabilidades**.

Estructura inicial propuesta:

```text
src/
├── app/
│   ├── (auth)/
│   │   ├── login/
│   │   │   └── page.tsx
│   │   ├── register/
│   │   │   └── page.tsx
│   │   └── layout.tsx
│   │
│   ├── (private)/
│   │   ├── dashboard/
│   │   │   └── page.tsx
│   │   ├── properties/
│   │   │   ├── page.tsx
│   │   │   ├── new/
│   │   │   └── [id]/
│   │   ├── leads/
│   │   │   ├── page.tsx
│   │   │   ├── new/
│   │   │   └── [id]/
│   │   ├── visits/
│   │   │   └── page.tsx
│   │   │   ├── new/
│   │   │   └── [id]/
│   │   ├── layout.tsx
│   │   └── loading.tsx
│   │
│   └── api/
│       ├── auth/
│       │   ├── register/
│       │   │   └── route.ts
│       │   ├── login/
│       │   │   └── route.ts
│       │   ├── logout/
│       │   │   └── route.ts
│       │   └── me/
│       │       └── route.ts
│       ├── properties/
│       │   ├── route.ts
│       │   └── [id]/
│       ├── leads/
│       │   ├── route.ts
│       │   └── [id]/
│       └── visits/
│       	  ├── route.ts
│           └── [id]/
│
├── features/
│   ├── auth/
│   │   ├── components/
│   │   ├── schemas/
│   │   ├── types/
│   │   ├── utils/
│   │   └── server/
│   │       ├── services/
│   │       ├── repositories/
│   │
│   ├── properties/
│   │   ├── components/
│   │   ├── schemas/
│   │   ├── types/
│   │   ├── hooks/
│   │   └── server/
│   │       ├── services/
│   │       ├── repositories/
│   │       └── utils/
│   │
│   ├── leads/
│   │   ├── components/
│   │   ├── schemas/
│   │   ├── types/
│   │   ├── hooks/
│   │   └── server/
│   │       ├── services/
│   │       ├── repositories/
│   │       └── utils/
│   │
│   ├── visits/
│   │   ├── components/
│   │   ├── schemas/
│   │   ├── types/
│   │   ├── hooks/
│   │   └── server/
│   │       ├── services/
│   │       ├── repositories/
│   │       └── utils/
│   │
│   └── dashboard/
│       ├── components/
│       ├── hooks/
│       └── types/
│
├── entities/
│   ├── global/
│   └── tenant/
│
├── lib/
│   ├── db/
│   ├── auth/
│   ├── tenancy/
│   ├── idempotency/
│   ├── rateLimit/
│   └── http/
│
├── shared/
│   ├── components/
│   ├── hooks/
│   ├── utils/
│   ├── types/
│   └── constants/
│
└── tests/
    ├── unit/
    ├── integration/
    └── e2e/
```
Adapta la estructura si TypeORM o Next.js requieren una organización más apropiada, pero explícame cualquier cambio. No crees archivos vacíos o abstracciones innecesarias.

Responsabilidades:

- `app/`: páginas y endpoints HTTP de Next.js.
- `features/`: lógica de cada funcionalidad inmobiliaria, con sus componentes, servicios, tipos, utilidades y esquemas de validación.
- `entities/`: entidades TypeORM, inicialmente para las tablas globales.
- `lib/db/`: conexión a PostgreSQL y migraciones.
- `lib/auth/`: autenticación, sesiones y autorización.
- `lib/tenancy/`: resolución segura del tenant y acceso al schema correcto.
- `lib/idempotency/`: almacenamiento y verificación de claves idempotentes.
- `lib/rate-limit/`: control de solicitudes.
- `lib/http/` : Centraliza las utilidades HTTP para estandarizar respuestas, manejar errores y códigos de estado. Evita duplicar código en los Route Handlers.
- `shared/`: componentes, tipos y utilidades verdaderamente compartidos.
- `tests/`: pruebas automatizadas.

### Organización de las features y carpeta `server/`

El proyecto debe organizarse combinando una arquitectura **por funcionalidades (feature-based)** y una arquitectura **por capas (layered architecture)**.

Cada feature agrupa los elementos relacionados con su funcionalidad. Por ejemplo, `features/properties/` contiene la interfaz, esquemas de validación, tipos, hooks y lógica de servidor relacionados con la gestión de propiedades.

Dentro de cada feature debe existir una carpeta `server/`, que agrupe exclusivamente el código que se ejecuta en el servidor. Su estructura será:

```text
features/
└── nombreFeature/
    ├── components/
    ├── schemas/
    ├── types/
    ├── hooks/
    └── server/
        ├── services/
        ├── repositories/
        └── utils/
```

Responsabilidades:

- **`server/services/`**: contiene la lógica de negocio. Los servicios coordinan las operaciones, aplican reglas del dominio y utilizan los repositorios. No deben encargarse de construir respuestas HTTP.
- **`server/repositories/`**: contiene la lógica de acceso y persistencia de datos mediante TypeORM. Se encarga de consultar, crear, actualizar y eliminar registros en PostgreSQL. Para las entidades del dominio, debe trabajar sobre el schema correspondiente al tenant autenticado.
- **`server/utils/`**: contiene funciones auxiliares específicas del servidor y de esa feature. No debe utilizarse para lógica que pertenezca claramente a un service o repository.

Los endpoints HTTP se mantienen en `src/app/api/`, mediante Next.js Route Handlers (`route.ts`). Estos actúan como la capa de entrada: reciben la petición, validan la sesión, resuelven el tenant, aplican controles como rate limiting e idempotencia, validan el body con Zod y delegan la lógica al service correspondiente.

El flujo general será:

`Route Handler → Service → Repository → TypeORM → PostgreSQL`

La organización por features permite mantener cada módulo funcional independiente, mientras que la separación por capas hace que las responsabilidades estén claras y facilita las pruebas.

La carpeta `server/` debe mantenerse aislada del código cliente. Los componentes React, hooks de cliente y páginas no deben importar repositorios, entidades de TypeORM, servicios que accedan directamente a la base de datos ni variables de entorno privadas. Las llamadas desde el cliente deben realizarse a través de los endpoints HTTP.

Los servicios y repositorios que utilicen credenciales o APIs privadas deben marcarse con `import "server-only"` cuando corresponda, para evitar que sean incluidos accidentalmente en el bundle del navegador.

No mezcles componentes de cliente con módulos de acceso a la base de datos. Las credenciales y la lógica sensible deben permanecer en el servidor.



## 4\. Requisitos obligatorios

### Multitenancy

- Las tablas globales estarán en `public`.
- Cada tenant tendrá su propio schema PostgreSQL con las tablas de propiedades, interesados y visitas.
- Al registrar una inmobiliaria, crear su schema y las tablas necesarias automáticamente.
- Resolver el tenant a partir de la sesión autenticada y los datos del servidor.
- Nunca aceptar un nombre de schema sin validar desde el navegador ni concatenar entrada no confiable en SQL.
- Demostrar aislamiento mediante pruebas con al menos dos tenants.
- Evitar estados parciales al crear un tenant y sus tablas.

### Autenticación y seguridad

- Registro e inicio de sesión.
- Hash de contraseñas.
- JWT en cookie `httpOnly` y configuración segura de cookies.
- Protección de rutas y autorización del tenant.
- Validación de entradas en el backend mediante Zod.
- No exponer secretos ni credenciales.

### CRUD

Implementar operaciones de creación, consulta, actualización y eliminación de propiedades, interesados y visitas.

### Idempotencia

- Implementar `Idempotency-Key` en al menos dos operaciones de escritura.
- Repetir una petición con la misma clave y el mismo payload debe devolver el resultado original sin duplicar datos.
- Definir qué sucede cuando se reutiliza una clave con un payload diferente.
- Manejar correctamente concurrencia y persistencia del resultado.
- Documentar la duración de las claves y la estrategia elegida.

### Rate limiting

- Aplicar límites como mínimo al login y a endpoints de creación.
- Al superar el límite, responder HTTP `429` con `Retry-After`.
- Utilizar almacenamiento compartido y operaciones seguras frente a concurrencia, evitando depender únicamente de memoria local.
- Documentar la estrategia y sus límites.

### Renderizado

Como se usa Next.js, implementar y justificar al menos dos estrategias entre SSR, CSR y SSG, según el enunciado. Una opción inicial es SSG para una página pública y SSR o CSR para el panel privado.

### Docker, CI/CD y documentación

- Dockerfile y `docker-compose.yml` que permitan levantar la aplicación con un comando.
- `.env.example` sin secretos.
- GitHub Actions que instale dependencias y ejecute lint, build o pruebas.
- README con nicho, arquitectura, instrucciones de ejecución, resolución del tenant, idempotencia y rate limiting.

## 5\. Cronograma acordado

Día 1: Supabase, Next.js, TypeORM, conexión y prueba de schemas dinámicos.

Día 2: registro, login, hash de contraseñas, JWT y protección de rutas.

Día 3: creación automática de schemas y tablas por tenant, más pruebas de aislamiento.

Día 4: CRUD de las tres entidades y validación con Zod.

Día 5: idempotencia y rate limiting.

Día 6: pruebas, renderizado y README.

Día 7: Docker, Docker Compose, variables de ejemplo, CI/CD y ensayo de sustentación.

La prioridad es desarrollar primero el núcleo funcional y dejar Docker y CI/CD para el final. Si el tiempo no alcanza, prioriza los requisitos obligatorios antes de añadir funcionalidades extras.

## 6\. Cómo quiero que trabajes conmigo

1. Guíame paso a paso, comenzando por el día 1.
2. Da comandos concretos y explica desde qué carpeta ejecutarlos.
3. Muestra el contenido completo de los archivos cuando llegue el momento de crearlos o modificarlos.
4. Explica para qué sirve cada archivo y cómo encaja en la arquitectura.
5. No asumas que las variables de entorno están configuradas ni que los comandos han funcionado: pídeme el resultado cuando sea necesario.
6. No pases al siguiente bloque hasta verificar el anterior cuando haya dependencias técnicas importantes.
7. Evita complejidad innecesaria; tengo experiencia básica con ORM y migraciones.
8. No me pidas publicar contraseñas, tokens ni cadenas de conexión. Usa nombres de variables y valores ficticios en los ejemplos.
9. No inventes resultados de pruebas ni afirmes que algo funciona sin verificarlo.
10. Ayúdame a preparar explicaciones que pueda utilizar durante la sustentación.
