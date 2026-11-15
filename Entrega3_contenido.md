# Profile Manager (Magneto) — Entrega 3: Producto funcional

> **Respaldo de contenido (Opción B).** Este archivo tiene el mismo contenido que `Entrega3_Final_LLENO.docx`, en Markdown, por si prefieres copiar y pegar las secciones directamente en la plantilla original de Word (conservando sus estilos y tablas). Los bloques `[PENDIENTE EQUIPO: ...]` los completa el equipo.

## Portada

- **Universidad EAFIT**
- **Profile Manager (Magneto)** — Entrega 3: Producto funcional
- **Integrantes:** Viliam Mesa, Juan Pablo Marín, Matías Zapata, Samuel Valencia
- **Repositorios:** backend `github.com/magneto-team-eafit/magneto-backend` · frontend `github.com/magneto-team-eafit/magneto-frontend`
- [PENDIENTE EQUIPO: fecha de entrega y nombre del curso/profesor]

---

## Sección 1: Aspectos generales de la entrega

### Introducción

Muchos candidatos pierden oportunidades laborales porque su información está dispersa, no saben qué tan completo está su perfil frente a lo que piden las empresas, y no tienen una forma ordenada de seguir el avance de sus postulaciones. Profile Manager es una plataforma web para Magneto que aborda ese problema de principio a fin: el candidato se registra, construye su perfil guiado por un asistente por pasos, explora un catálogo de vacantes con búsqueda y filtros, recibe recomendaciones con una explicación del porqué, se postula y hace seguimiento a cada proceso en un tablero tipo Kanban con notificaciones y registro de actividad.

La solución se construyó bajo un principio de división por verticales de negocio: en lugar de separar el trabajo por capas (uno hace backend, otro frontend), cada integrante es dueño de una vertical completa (su base de datos, sus endpoints y sus pantallas), atravesando las cuatro capas de Clean Architecture dentro de su propio dominio. Las cuatro verticales son: A) Perfil y Autenticación, B) Vacantes e Ingesta, C) Recomendación y Matching, y D) Postulación y Trazabilidad.

Este documento describe la arquitectura de la solución, los principios de diseño (SOLID, GRASP y Clean Code) y patrones aplicados con ejemplos del código real, la funcionalidad implementada en este sprint y las conclusiones del equipo.

## Sección 2: Evaluación Sprint anterior

[PENDIENTE EQUIPO: evidencias de la retrospectiva, técnica usada (p. ej. Start/Stop/Continue), acciones de mejora y evidencias de daily/weekly.]

Resumen técnico del sprint anterior (Entrega 2):

- Vertical A: wizard de perfil por pasos con validación y barra de completitud dinámica.
- Vertical B: ingesta del dataset re-ejecutable, búsqueda full-text e índice GIN, filtros avanzados y detalle de vacante.
- Vertical C: scoring mejorado con historial de recomendaciones y explicación del puntaje.
- Vertical D: estados adicionales (en revisión, entrevista, descartado) con transición de estado, notificaciones in-app y registro de actividad; tablero Kanban con varias columnas.

## Sección 3: Planificación Sprint actual

[PENDIENTE EQUIPO: Historias de Usuario priorizadas, evidencias del planning con poker planning, estado del Sprint Backlog en GitHub Projects y su enlace, capturas.]

Historias de usuario de este sprint (Entrega 3):

- Como usuario quiero que mis postulaciones y notificaciones sean privadas, para que nadie más pueda verlas o modificarlas (seguridad, vertical D).
- Como usuario quiero un resumen de mi tablero (total, por estado y tasa de avance), para medir mi progreso (vertical D).
- Como usuario quiero arrastrar las tarjetas entre columnas, para cambiar el estado de forma más natural (vertical D).
- Como equipo queremos pruebas automatizadas y CI, para asegurar la calidad en cada cambio (transversal).
- Como usuario quiero recomendaciones mejor explicadas y un catálogo pulido (verticales C y B).

## Sección 4: Aspectos estructurales y arquitectónicos de la solución

### Arquitectura propuesta

El sistema sigue una arquitectura limpia (Clean Architecture) en cuatro capas concéntricas, sobre Express + TypeScript (backend) y React + Vite (frontend). De adentro hacia afuera:

- **Dominio (domain):** entidades y reglas puras (Usuario, Perfil, Vacante) e interfaces/puertos (IPostulacionRepository, IUsuarioRepository, IHashService, IJwtService). No depende de frameworks ni de la base de datos.
- **Aplicación (application):** casos de uso (RegistrarUsuario, GuardarPasoPerfil, PostulacionUseCase, ListarVacantes). Dependen de interfaces, no de implementaciones.
- **Infraestructura (infrastructure):** implementaciones concretas (PrismaPostulacionRepository, PrismaUsuarioRepository, BcryptHashService, JwtService) y el cliente de base de datos.
- **Interfaces (interfaces):** entrega HTTP: controladores, rutas de Express y middlewares (authMiddleware, soloPropietario).

La dependencia siempre apunta hacia adentro (regla de dependencia), lograda con inversión de dependencias: los casos de uso reciben las interfaces por el constructor y en el arranque se inyecta la implementación con Prisma. Cada vertical replica estas cuatro capas en su dominio. El frontend consume la API por HTTP con el token JWT en el header Authorization.

### Resumen de la arquitectura

| Aspecto | Descripción |
|---|---|
| Tipo de aplicación | Web (SPA en React servida por Vite + API REST en Node/Express). |
| Estilo arquitectónico | Layered, Domain Driven Design, Client/Server, Object-Oriented, Component-Based. |
| Lenguaje | TypeScript (backend y frontend); SQL. |
| Aspectos técnicos | PostgreSQL + Prisma; JWT + bcrypt; búsqueda full-text (GIN); CI con GitHub Actions; OpenAPI/Swagger. |
| Frameworks | Backend: Express 5, Prisma 7, jsonwebtoken, bcrypt. Frontend: React 19, Vite, React Router, Tailwind CSS 4. |

### Vista Lógica - Diagrama de Clases

[PENDIENTE EQUIPO: insertar imagen.] Contenido esperado (vertical D):

- Interfaz `IPostulacionRepository`: crear, obtenerPorUsuario, obtenerPorId, cambiarEstado, obtenerActividad, contarPorEstado, obtenerNotificaciones, marcarNotificacionLeida.
- `PostulacionUseCase` depende de `IPostulacionRepository`.
- `PrismaPostulacionRepository` implementa la interfaz con Prisma.
- Rutas/controladores que invocan el caso de uso y aplican authMiddleware.
- Errores de dominio: ErrorPostulacionDuplicada, ErrorPostulacionNoEncontrada, ErrorAccesoDenegado, ErrorEstadoInvalido, ErrorRecursoNoEncontrado.

### Vista Física - Diagrama de Despliegue

[PENDIENTE EQUIPO: insertar imagen.] Nodos: Cliente (navegador, React/Vite), Servidor de aplicación (API Express/Node), Base de datos (PostgreSQL). Comunicación: navegador ⇄ API por HTTP/JSON (JWT); API ⇄ PostgreSQL por Prisma.

### Persistencia

Motor: PostgreSQL con ORM Prisma. Se eligió PostgreSQL por ser relacional, robusto, open source y con búsqueda full-text nativa (índice GIN). Prisma aporta esquema declarativo, migraciones versionadas y cliente tipado, manteniendo el acceso a datos en infraestructura. Entidades: Usuario, Perfil (Educacion, Experiencia, Habilidad), Vacante, Postulacion, RegistroActividad, Notificacion, HistorialRecomendacion. Reglas como "un usuario no puede postularse dos veces a la misma vacante" se garantizan con restricciones únicas en la base de datos.

## Sección 5: Principios de diseño

### 5.1. Principios SOLID

| Principio | Aplicación en el proyecto |
|---|---|
| Single Responsibility | Cada clase tiene una sola razón para cambiar: RegistrarUsuario solo orquesta; validación, hash y persistencia en clases separadas. |
| Open/Closed | Estados en una lista (ESTADOS_POSTULACION); agregar uno no obliga a reescribir la lógica; nuevas implementaciones de repositorio sin tocar casos de uso. |
| Liskov Substitution | FakePostulacionRepository y PrismaPostulacionRepository implementan IPostulacionRepository; el caso de uso funciona igual con cualquiera. |
| Interface Segregation | Interfaces pequeñas: IHashService (hashear/comparar), IJwtService (generar/verificar). |
| Dependency Inversion | PostulacionUseCase recibe IPostulacionRepository por constructor; las implementaciones Prisma/bcrypt se inyectan en interfaces. |

### 5.2. Patrones GRASP

| Patrón | Aplicación |
|---|---|
| Bajo Acoplamiento | Las capas se comunican por interfaces; la aplicación no conoce Prisma ni Express. |
| Alta Cohesión | Validación en validadores.ts, scoring en scoring.service.ts, datos en repositorios. |
| Experto | Perfil calcula su completitud y sugerencias porque tiene los datos. |
| Creador | Fábricas estáticas: Usuario.crear, Vacante.crear. |
| Controlador | Los controladores de Express delegan en los casos de uso. |
| Polimorfismo | Implementaciones de un puerto (Prisma vs. memoria) intercambiables. |
| Fabricación Pura | BcryptHashService, JwtService, servicios de scoring. |
| Indirección | Interfaces de repositorio como intermediario aplicación ⇄ datos. |
| Variaciones Protegidas | Al depender de interfaces, un cambio de tecnología queda aislado en infraestructura. |

### 5.3. Clean Code

- Nombres intención-reveladores: cambiarEstado, obtenerResumen, contarPorEstado, esEstadoValido.
- Funciones pequeñas; validaciones separadas de la orquestación.
- Errores de dominio (ErrorAccesoDenegado, ErrorPostulacionDuplicada) traducidos a HTTP (403, 409, 404), sin exponer errores crudos de la base de datos.
- Comentarios que explican el porqué, no el qué.
- Sin números mágicos ni duplicación: estados y etiquetas centralizados en estados.ts.

## Sección 6: Patrones de diseño

### Patrón 1: Repository
- **Clasificación:** Estructural (acceso a datos / DDD).
- **Intención:** separar la lógica de negocio del acceso a datos.
- **Aplicabilidad:** aislar la persistencia para cambiar el motor o probar sin base de datos real.
- **Estructura:** [PENDIENTE EQUIPO: diagrama].
- **Participantes:** IPostulacionRepository, PrismaPostulacionRepository, FakePostulacionRepository.
- **Colaboraciones:** los casos de uso piden datos por la interfaz; la implementación traduce a Prisma/SQL.
- **Consecuencias:** + desacopla negocio y datos, facilita pruebas; − una indirección más.
- **Implementación:** interfaz en dominio, implementación en infraestructura, inyectada por constructor.
- **Código:** `src/domain/postulaciones/repositories/IPostulacionRepository.ts` y `src/infrastructure/repositories/PrismaPostulacionRepository.ts`.
- **Usos conocidos:** Spring Data, Entity Framework, Hibernate.
- **Relacionados:** Dependency Injection, Unit of Work.

### Patrón 2: Dependency Injection
- **Clasificación:** Creación (Inversión de Control).
- **Intención:** entregar al objeto sus dependencias desde afuera.
- **Aplicabilidad:** colaboradores que varían o se sustituyen en pruebas.
- **Estructura:** [PENDIENTE EQUIPO: diagrama].
- **Participantes:** cliente (PostulacionUseCase, RegistrarUsuario), dependencias abstractas (IPostulacionRepository, IHashService), ensamblador (rutas/arranque).
- **Colaboraciones:** el ensamblador crea la implementación y la pasa por constructor; el cliente la usa por la interfaz.
- **Consecuencias:** + desacoplamiento y testabilidad; − gestionar el ensamblado.
- **Implementación:** `constructor(private repo: IPostulacionRepository)`.
- **Código:** `src/application/use-cases/PostulacionUseCase.ts`, `src/application/perfil/useCases/RegistrarUsuario.ts`.
- **Usos conocidos:** Angular, NestJS, Spring.
- **Relacionados:** Repository, Strategy, Factory.

### Patrón 3: Factory Method
- **Clasificación:** Creación.
- **Intención:** encapsular la creación de objetos del dominio garantizando estado válido.
- **Aplicabilidad:** crear requiere validación; distinguir "crear nuevo" de "reconstruir desde la base de datos".
- **Estructura:** [PENDIENTE EQUIPO: diagrama].
- **Participantes:** Usuario, Vacante, Perfil con `crear()` y `reconstruir()`.
- **Colaboraciones:** `Usuario.crear({...})` valida y devuelve instancia consistente; `reconstruir()` rehidrata sin revalidar.
- **Consecuencias:** + creación controlada, invariantes; − más métodos en la entidad.
- **Implementación:** métodos estáticos con constructor privado.
- **Código:** `src/domain/perfil/entities/Usuario.ts`, `src/domain/vacantes/entities/Vacante.ts`.
- **Usos conocidos:** Calendar.getInstance(), ORMs.
- **Relacionados:** Builder, Abstract Factory.

### Patrón 4: Middleware / Chain of Responsibility
- **Clasificación:** Comportamiento.
- **Intención:** procesar una petición por una cadena de manejadores.
- **Aplicabilidad:** controles transversales: autenticación, autorización, rate limiting.
- **Estructura:** [PENDIENTE EQUIPO: diagrama].
- **Participantes:** authMiddleware, soloPropietario, limitadorIntentos.
- **Colaboraciones:** Express ejecuta los middlewares en orden; cada uno llama a next() o corta con error (401/403/429).
- **Consecuencias:** + separa preocupaciones transversales, reutilizable; − el orden importa.
- **Implementación:** funciones (req, res, next) montadas con router.use(...).
- **Código:** `src/interfaces/perfil/middlewares/authMiddleware.ts`, `soloPropietario.ts`, `limitadorIntentos.ts`.
- **Usos conocidos:** Express, servlets, ASP.NET.
- **Relacionados:** Decorator, Command.

## Sección 7: Funcionalidad y demostración

Árbol de directorios del backend (una carpeta por capa, cada vertical dentro):

```
src/
  domain/          # entidades, interfaces (puertos), errores de negocio
    perfil/  vacantes/  postulaciones/
  application/     # casos de uso
    perfil/  vacantes/  use-cases/ (PostulacionUseCase)
  infrastructure/  # Prisma, repositorios, servicios (bcrypt, jwt)
  interfaces/      # rutas, controladores, middlewares (HTTP)
  recommendation/  # vertical C con sus propias capas
  __tests__/       # pruebas e2e del flujo de postulaciones
```

**Historias de usuario demostradas (mínimo 3):**

1. **Catálogo de vacantes con acción por registro (B):** la vista de Vacantes lista registros de la tabla `vacantes` (GET /vacantes con paginación, búsqueda y filtros); cada registro ofrece "Postularme" y enlace al detalle.
2. **Formulario que manipula datos: postularse y mover de estado (D):** POST /postulaciones (inserción); en el tablero, arrastrar o botones disparan PATCH /postulaciones/:id/estado (actualización), que además genera notificación y actividad en la misma transacción. Postularse dos veces → 409 con mensaje claro.
3. **Perfil por pasos con completitud (A):** el wizard guarda cada paso (PUT /perfil/paso/:paso, actualización) y recalcula la completitud con sugerencias.

**Acceso a datos y seguridad:** todas las rutas de postulaciones y notificaciones exigen token JWT; el usuarioId se deduce del token (no de la URL) y cada operación verifica que la postulación sea del usuario autenticado. El flujo principal está cubierto por pruebas e2e automatizadas, y la CI (GitHub Actions) corre lint, build y pruebas en cada Pull Request.

[PENDIENTE EQUIPO: insertar capturas de pantalla de cada historia.]

## Conclusiones y lecciones aprendidas

- Dividir por verticales permitió full-stack real y avance integral por entrega.
- Clean Architecture e inversión de dependencias facilitaron las pruebas (flujo completo con repositorio en memoria, sin base de datos).
- Traducir errores de dominio a HTTP (403/404/409) mejoró seguridad y experiencia.
- La seguridad no debe dejarse para el final: deducir el usuario del token cerró una vulnerabilidad de acceso a datos ajenos.
- CI y revisión cruzada de PRs elevaron la calidad y evitaron romper main.
- [PENDIENTE EQUIPO: dificultades concretas del sprint y cómo se resolvieron.]

## Referencias

[PENDIENTE EQUIPO: formato APA/IEEE.]

- Martin, R. C. (2017). *Clean Architecture*. Prentice Hall.
- Larman, C. (2004). *Applying UML and Patterns* (GRASP). Prentice Hall.
- Gamma, E. et al. (1994). *Design Patterns* (GoF). Addison-Wesley.
- Documentación oficial: Express, Prisma, React, PostgreSQL.
