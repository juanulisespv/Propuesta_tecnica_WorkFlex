# Stack Tecnológico WorkFlex

## Frontend

### Web Application
**Framework**: React 18 + TypeScript
- **Justificación**: 
  - Ecosistema maduro y amplia comunidad
  - TypeScript para type safety y mejor developer experience
  - Componentes reutilizables
  - Excelente tooling y documentación

**Build Tool**: Vite
- Desarrollo más rápido que Create React App
- Hot Module Replacement (HMR) instantáneo
- Bundle optimization automático

**Gestión de Estado**: Redux Toolkit + RTK Query
- Redux Toolkit: Menos boilerplate que Redux tradicional
- RTK Query: Caché automático de datos de API
- DevTools para debugging

**Routing**: React Router v6
- Navegación declarativa
- Code splitting por rutas
- Protected routes fáciles de implementar

**UI Components**: 
- **Tailwind CSS**: Utility-first, diseño rápido, bundle pequeño
- **Headless UI**: Componentes accesibles sin estilos
- **Heroicons**: Iconos consistentes

**Forms**: React Hook Form + Zod
- React Hook Form: Performance optimizado, menos re-renders
- Zod: Validación de schemas con TypeScript inference

**HTTP Client**: Axios
- Interceptors para autenticación
- Manejo automático de errores
- Request/response transformation

---

## Backend

### API Server
**Framework**: Node.js + Express.js (o alternativa: Python + FastAPI)

#### Opción A: Node.js + Express.js
**Justificación**:
- ✅ Mismo lenguaje que frontend (JavaScript/TypeScript)
- ✅ Excelente para I/O operations (muchas llamadas a BD/APIs)
- ✅ Ecosistema NPM enorme
- ✅ Equipo puede trabajar full-stack

**Estructura**:
```
src/
├── config/           # Configuración (DB, Redis, APIs)
├── modules/
│   ├── auth/
│   │   ├── controllers/
│   │   ├── services/
│   │   ├── repositories/
│   │   ├── models/
│   │   └── routes.ts
│   ├── reservations/
│   ├── spaces/
│   └── payments/
├── shared/
│   ├── middleware/   # Auth, validation, error handling
│   ├── utils/
│   └── types/
├── database/
│   ├── migrations/
│   └── seeds/
└── app.ts           # Entry point
```

**Librerías clave**:
- **express**: Web framework
- **express-validator**: Validación de inputs
- **helmet**: Security headers
- **cors**: Cross-origin configuration
- **compression**: Gzip responses
- **morgan**: HTTP logging
- **dotenv**: Environment variables

#### Opción B: Python + FastAPI (alternativa)
**Ventajas**:
- Mejor para procesamiento de reportes complejos
- Type hints nativos
- Auto-documentación OpenAPI
- Performance comparable a Node.js

**Elección**: **Node.js + Express.js** 
- Razón: Equipo puede ser full-stack JavaScript, menor curva de aprendizaje

---

## Base de Datos

### Base de Datos Principal: PostgreSQL 15
**Justificación**:
- ✅ ACID compliance (crítico para transacciones de pago)
- ✅ JSON/JSONB support (flexibilidad para datos no estructurados)
- ✅ Excelente performance en reads y writes
- ✅ Extensiones potentes (PostGIS para geolocalización)
- ✅ Open source, sin vendor lock-in
- ✅ Replicación y alta disponibilidad

**Alternativas descartadas**:
- ❌ MySQL: Menor soporte de JSON, menos features avanzados
- ❌ MongoDB: No relacional, peor para transacciones complejas
- ❌ SQL Server: Licencias costosas

**ORM**: Prisma
- Type-safe query builder
- Auto-migrations
- Excelente developer experience
- Generación automática de tipos TypeScript

```typescript
// Ejemplo de uso Prisma
const reserva = await prisma.reserva.create({
  data: {
    clienteId: userId,
    espacioId: spaceId,
    fechaReserva: date,
    estado: 'CONFIRMADA'
  },
  include: {
    espacio: true,
    cliente: true
  }
});
```

---

## Cache Layer

### Redis 7
**Justificación**:
- ✅ In-memory store ultra-rápido
- ✅ Múltiples usos: cache, sessions, rate limiting
- ✅ Pub/Sub para notificaciones real-time
- ✅ TTL automático para expiración de caché

**Casos de uso en WorkFlex**:
1. **Session store**: JWT tokens blacklist
2. **Query cache**: Listados de espacios (TTL: 5 minutos)
3. **Disponibilidad cache**: Estado de ocupación (TTL: 1 minuto)
4. **Rate limiting**: Prevenir abuso de API
5. **Queue de notificaciones**: Emails/SMS pendientes

**Librería**: ioredis (Node.js)

---

## Autenticación y Autorización

### JWT (JSON Web Tokens) + Refresh Tokens
**Flow**:
1. Login → Genera access token (15 min) + refresh token (7 días)
2. Access token en header de cada request
3. Cuando expira → Usa refresh token para renovar
4. Logout → Blacklist tokens en Redis

**Librería**: jsonwebtoken

**Hashing de passwords**: bcrypt (factor 12)

**OAuth 2.0** (Fase 2):
- Google Sign-In
- Microsoft/LinkedIn (para usuarios corporativos)

**Librería OAuth**: Passport.js

---

## Integraciones Externas

### 1. Pagos: Stripe
**Justificación**:
- API más developer-friendly del mercado
- Compliance PCI-DSS automático
- Múltiples métodos de pago (tarjeta, SEPA, wallets)
- Webhooks para sincronización de estados
- Test mode robusto

**Alternativa europea**: Redsys (si WorkFlex necesita compliance español específico)

**SDK**: @stripe/stripe-js

### 2. Emails: SendGrid
**Justificación**:
- 100 emails/día gratuitos
- Templates HTML potentes
- Tracking de opens/clicks
- API simple

**Alternativa**: Amazon SES (más económico a escala)

### 3. SMS: Twilio
**Para**:
- Recordatorios de reserva
- Códigos de verificación 2FA
- Alertas urgentes

### 4. Geolocalización: Google Maps API
**Para**:
- Autocomplete de direcciones
- Mapa en detalle de sede
- Cálculo de rutas

### 5. Storage: AWS S3
**Para**:
- Fotos de espacios
- Facturas en PDF
- Exports de reportes

**Alternativa**: Cloudflare R2 (sin costos de egress)

---

## Monitoring y Observabilidad

### Logs: Winston (Node.js)
- Logs estructurados en JSON
- Múltiples transports (file, console, external)
- Niveles: error, warn, info, debug

### Metrics: Prometheus + Grafana
- Métricas de aplicación (requests/seg, latency, errores)
- Métricas de infraestructura (CPU, RAM, disco)
- Alertas automáticas (Slack/email)

### Error Tracking: Sentry
- Captura de excepciones no manejadas
- Source maps para debugging
- User context en errores
- Alertas en tiempo real

### APM (Application Performance Monitoring): New Relic o DataDog (Fase 2)
- Tracing distribuido
- Análisis de queries lentos
- User experience monitoring

---

## CI/CD Pipeline

### Source Control: GitHub
- Branching strategy: Git Flow
- Pull Requests con code review obligatorio
- Branch protection rules

### CI/CD: GitHub Actions
**Pipeline**:
```yaml
1. Push a dev → 
   - Lint (ESLint)
   - Type check (TypeScript)
   - Unit tests (Jest)
   - Build

2. Pull Request a main →
   - Lo anterior +
   - Integration tests
   - E2E tests (Playwright)
   - Security scan (Snyk)

3. Merge a main →
   - Deploy a staging
   - Smoke tests
   - Manual approval
   - Deploy a production
```

### Containerization: Docker
**Dockerfile multi-stage**:
```dockerfile
# Build stage
FROM node:18-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# Production stage
FROM node:18-alpine
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
EXPOSE 3000
CMD ["node", "dist/app.js"]
```

### Orchestration (Fase 2): Docker Compose → Kubernetes
- Inicialmente: Docker Compose en VPS
- Cuando escale: AWS ECS o Kubernetes

---

## Infraestructura

### Hosting (Fase 1 - MVP)
**Opción recomendada**: DigitalOcean Droplet
- 2 vCPU, 4GB RAM: $24/mes
- PostgreSQL managed: $15/mes
- Redis managed: $15/mes
- S3-compatible Spaces: $5/mes
- **Total: ~$60/mes**

**Alternativas**:
- AWS Lightsail: Similar precio, menos features
- Railway: $20/mes, muy simple (good for prototyping)
- Render: Auto-scaling, más caro ($25+/mes)

### Hosting (Fase 2 - Producción)
**Opción escalable**: AWS
- ECS Fargate: Containers sin gestionar servidores
- RDS PostgreSQL: Multi-AZ, backups automáticos
- ElastiCache Redis: Replicación, failover
- S3: Storage ilimitado
- CloudFront CDN: Assets globales
- **Costo estimado: $200-400/mes** (depende de tráfico)

---

## Testing

### Unit Tests: Jest + Testing Library
- Cobertura objetivo: > 80%
- Tests para services y repositories
- Mocking de dependencias externas

### Integration Tests: Supertest
- Tests de endpoints completos
- Base de datos de test (Docker container)

### E2E Tests: Playwright
- Flujos críticos:
  - Registro → Login → Reserva → Pago
  - Dashboard admin
- Ejecución en CI antes de deploy a producción

### Load Testing: k6 o Artillery
- Simular 1000 usuarios concurrentes
- Identificar cuellos de botella antes de lanzamiento

---

## Security

### Medidas implementadas:
1. **HTTPS only**: SSL certificate (Let's Encrypt)
2. **Helmet.js**: Security headers (CSP, XSS protection)
3. **Rate limiting**: Express-rate-limit
4. **SQL Injection prevention**: Prisma parameterized queries
5. **XSS prevention**: DOMPurify en frontend
6. **CSRF tokens**: csurf middleware
7. **Secrets management**: AWS Secrets Manager o Vault (Fase 2)
8. **Dependency scanning**: Snyk en CI/CD
9. **API authentication**: JWT + Refresh tokens
10. **Input validation**: Zod schemas en backend

---

## Resumen del Stack

| Categoría | Tecnología | Justificación |
|-----------|-----------|---------------|
| **Frontend** | React + TypeScript + Tailwind | Ecosistema maduro, type safety, desarrollo rápido |
| **Backend** | Node.js + Express + TypeScript | Full-stack JS, I/O performance, comunidad |
| **Base de Datos** | PostgreSQL 15 | ACID, JSON support, open source |
| **Cache** | Redis 7 | In-memory, múltiples casos de uso |
| **ORM** | Prisma | Type-safe, migrations, DX excelente |
| **Autenticación** | JWT + bcrypt | Stateless, secure, estándar |
| **Pagos** | Stripe | Developer-friendly, compliance |
| **Email** | SendGrid | Gratuito para empezar, escalable |
| **Storage** | AWS S3 | Ilimitado, económico |
| **Monitoring** | Sentry + Prometheus/Grafana | Error tracking + métricas |
| **CI/CD** | GitHub Actions | Integrado, gratuito para open source |
| **Hosting MVP** | DigitalOcean | $60/mes, simple, escalable |

Este stack permite:
- ✅ Desarrollar MVP en 3-4 meses
- ✅ Escalar a 10,000+ usuarios sin reescribir
- ✅ Equipo puede trabajar full-stack
- ✅ Costos iniciales bajos ($60-100/mes)
- ✅ Path claro de evolución a microservicios si es necesario
