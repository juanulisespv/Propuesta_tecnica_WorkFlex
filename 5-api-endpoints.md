# API REST - WorkFlex

## Principios de Diseño de la API

### 1. Convenciones RESTful
- Uso de verbos HTTP estándar (GET, POST, PUT, PATCH, DELETE)
- URIs basados en recursos (sustantivos, no verbos)
- Códigos de estado HTTP apropiados
- Versionado de API: `/api/v1/`

### 2. Estructura de Respuestas
```json
{
  "success": true,
  "data": { /* payload */ },
  "message": "Operación exitosa",
  "metadata": {
    "timestamp": "2025-11-09T10:30:00Z",
    "version": "1.0.0"
  }
}
```

**En caso de error**:
```json
{
  "success": false,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Datos de entrada inválidos",
    "details": [
      {
        "field": "email",
        "message": "El email es inválido"
      }
    ]
  },
  "metadata": {
    "timestamp": "2025-11-09T10:30:00Z",
    "requestId": "abc123"
  }
}
```

### 3. Autenticación
- Header: `Authorization: Bearer <JWT_TOKEN>`
- Todos los endpoints (excepto login/registro) requieren autenticación
- Roles: `CLIENT`, `ADMIN`, `SUPER_ADMIN`

### 4. Rate Limiting
- **Usuarios no autenticados**: 100 requests/hora
- **Clientes autenticados**: 1000 requests/hora
- **Administradores**: 5000 requests/hora
- Header de respuesta: `X-RateLimit-Remaining: 950`

---

## Endpoints Detallados

## 1. Autenticación y Usuarios

### POST `/api/v1/auth/register`
**Descripción**: Registro de nuevo usuario

**Request Body**:
```json
{
  "email": "maria@example.com",
  "password": "SecurePass123!",
  "nombre": "María",
  "apellidos": "García López",
  "telefono": "+34612345678",
  "aceptaTerminos": true
}
```

**Response** (201 Created):
```json
{
  "success": true,
  "data": {
    "usuario": {
      "id": 123,
      "email": "maria@example.com",
      "nombre": "María García López",
      "rol": "CLIENT"
    },
    "accessToken": "eyJhbGc...",
    "refreshToken": "eyJhbGc...",
    "expiresIn": 900
  }
}
```

**Errores**:
- `400 BAD_REQUEST`: Email ya registrado
- `422 VALIDATION_ERROR`: Datos inválidos

---

### POST `/api/v1/auth/login`
**Descripción**: Inicio de sesión

**Request Body**:
```json
{
  "email": "maria@example.com",
  "password": "SecurePass123!"
}
```

**Response** (200 OK):
```json
{
  "success": true,
  "data": {
    "usuario": {
      "id": 123,
      "email": "maria@example.com",
      "nombre": "María García López",
      "rol": "CLIENT",
      "membresia": {
        "tipo": "PROFESIONAL",
        "activa": true,
        "creditosDisponibles": 10
      }
    },
    "accessToken": "eyJhbGc...",
    "refreshToken": "eyJhbGc...",
    "expiresIn": 900
  }
}
```

**Errores**:
- `401 UNAUTHORIZED`: Credenciales inválidas
- `403 FORBIDDEN`: Cuenta desactivada

---

### POST `/api/v1/auth/refresh`
**Descripción**: Renovar access token

**Request Body**:
```json
{
  "refreshToken": "eyJhbGc..."
}
```

**Response** (200 OK):
```json
{
  "success": true,
  "data": {
    "accessToken": "eyJhbGc...",
    "expiresIn": 900
  }
}
```

---

### GET `/api/v1/users/me`
**Descripción**: Obtener perfil del usuario actual

**Headers**: `Authorization: Bearer <token>`

**Response** (200 OK):
```json
{
  "success": true,
  "data": {
    "id": 123,
    "email": "maria@example.com",
    "nombre": "María",
    "apellidos": "García López",
    "telefono": "+34612345678",
    "fechaRegistro": "2025-01-15T10:00:00Z",
    "membresia": {
      "tipo": "PROFESIONAL",
      "fechaVencimiento": "2025-12-15",
      "creditosDisponibles": 10
    }
  }
}
```

---

### PATCH `/api/v1/users/me`
**Descripción**: Actualizar perfil

**Request Body**:
```json
{
  "nombre": "María José",
  "telefono": "+34699999999"
}
```

**Response** (200 OK): Usuario actualizado

---

## 2. Espacios

### GET `/api/v1/spaces`
**Descripción**: Listar espacios con filtros

**Query Parameters**:
- `sedeId` (opcional): Filtrar por sede
- `tipo` (opcional): PUESTO_COMPARTIDO | OFICINA_PRIVADA | SALA_REUNIONES
- `capacidadMin` (opcional): Capacidad mínima
- `precioMax` (opcional): Precio máximo por hora
- `fecha` (opcional): Fecha para verificar disponibilidad
- `page` (default: 1): Número de página
- `limit` (default: 20): Resultados por página

**Ejemplo**: `GET /api/v1/spaces?sedeId=1&tipo=SALA_REUNIONES&capacidadMin=6&page=1`

**Response** (200 OK):
```json
{
  "success": true,
  "data": {
    "espacios": [
      {
        "id": 45,
        "nombre": "Sala Innovation",
        "tipo": "SALA_REUNIONES",
        "capacidad": 8,
        "precioPorHora": 25.00,
        "sede": {
          "id": 1,
          "nombre": "WorkFlex Bilbao Centro",
          "direccion": "Gran Vía 45"
        },
        "equipamiento": ["Proyector", "Pizarra", "WiFi", "Aire acondicionado"],
        "fotos": ["url1.jpg", "url2.jpg"],
        "disponible": true
      }
    ],
    "pagination": {
      "page": 1,
      "limit": 20,
      "total": 45,
      "totalPages": 3
    }
  }
}
```

---

### GET `/api/v1/spaces/:id`
**Descripción**: Detalle de un espacio específico

**Response** (200 OK):
```json
{
  "success": true,
  "data": {
    "id": 45,
    "nombre": "Sala Innovation",
    "codigo": "BIL-SR-001",
    "tipo": "SALA_REUNIONES",
    "capacidad": 8,
    "precioPorHora": 25.00,
    "precioPorDia": 180.00,
    "superficieM2": 30,
    "descripcion": "Sala moderna perfecta para reuniones...",
    "equipamiento": ["Proyector 4K", "Pizarra digital", "WiFi 1Gbps"],
    "fotos": ["url1.jpg", "url2.jpg", "url3.jpg"],
    "sede": {
      "id": 1,
      "nombre": "WorkFlex Bilbao Centro",
      "direccion": "Gran Vía 45, 48001 Bilbao",
      "telefono": "+34944123456",
      "coordenadas": {
        "lat": 43.2627,
        "lng": -2.9253
      }
    },
    "accesible": true,
    "politicaCancelacion": "Cancelación gratuita hasta 24h antes"
  }
}
```

**Errores**:
- `404 NOT_FOUND`: Espacio no existe

---

### GET `/api/v1/spaces/:id/availability`
**Descripción**: Verificar disponibilidad de un espacio en una fecha

**Query Parameters**:
- `fecha` (requerido): Fecha en formato YYYY-MM-DD
- `duracion` (opcional, default: 1): Duración en horas

**Ejemplo**: `GET /api/v1/spaces/45/availability?fecha=2025-11-15`

**Response** (200 OK):
```json
{
  "success": true,
  "data": {
    "espacioId": 45,
    "fecha": "2025-11-15",
    "horariosDisponibles": [
      {
        "horaInicio": "08:00",
        "horaFin": "09:00",
        "disponible": true
      },
      {
        "horaInicio": "09:00",
        "horaFin": "10:00",
        "disponible": false,
        "reservadoPor": "Usuario A"
      },
      {
        "horaInicio": "10:00",
        "horaFin": "11:00",
        "disponible": true
      }
    ],
    "tasaOcupacion": 35.7
  }
}
```

---

## 3. Reservas

### POST `/api/v1/reservations`
**Descripción**: Crear una nueva reserva

**Headers**: `Authorization: Bearer <token>`

**Request Body**:
```json
{
  "espacioId": 45,
  "fechaReserva": "2025-11-15",
  "horaInicio": "10:00",
  "horaFin": "12:00"
}
```

**Response** (201 Created):
```json
{
  "success": true,
  "data": {
    "reserva": {
      "id": 789,
      "espacioId": 45,
      "espacioNombre": "Sala Innovation",
      "fechaReserva": "2025-11-15",
      "horaInicio": "10:00",
      "horaFin": "12:00",
      "duracion": 2,
      "precioTotal": 50.00,
      "estado": "PENDIENTE",
      "requierePago": true,
      "fechaCreacion": "2025-11-09T10:30:00Z"
    },
    "pagoId": 456,
    "checkoutUrl": "https://checkout.stripe.com/..."
  },
  "message": "Reserva creada. Completa el pago para confirmar."
}
```

**Errores**:
- `409 CONFLICT`: Horario no disponible
- `400 BAD_REQUEST`: Fecha en el pasado
- `403 FORBIDDEN`: Usuario sin créditos ni método de pago

---

### GET `/api/v1/reservations`
**Descripción**: Listar reservas del usuario actual

**Query Parameters**:
- `estado` (opcional): PENDIENTE | CONFIRMADA | COMPLETADA | CANCELADA
- `desde` (opcional): Fecha desde
- `hasta` (opcional): Fecha hasta
- `page`, `limit`

**Response** (200 OK):
```json
{
  "success": true,
  "data": {
    "reservas": [
      {
        "id": 789,
        "espacio": {
          "id": 45,
          "nombre": "Sala Innovation",
          "foto": "url.jpg"
        },
        "fecha": "2025-11-15",
        "horaInicio": "10:00",
        "horaFin": "12:00",
        "precioTotal": 50.00,
        "estado": "CONFIRMADA",
        "puedeModificar": true,
        "puedeCancelar": true
      }
    ],
    "pagination": { /* ... */ }
  }
}
```

---

### GET `/api/v1/reservations/:id`
**Descripción**: Detalle de una reserva específica

**Response** (200 OK):
```json
{
  "success": true,
  "data": {
    "id": 789,
    "espacio": {
      "id": 45,
      "nombre": "Sala Innovation",
      "direccion": "Gran Vía 45, Bilbao"
    },
    "fechaReserva": "2025-11-15",
    "horaInicio": "10:00",
    "horaFin": "12:00",
    "duracion": 2,
    "precioTotal": 50.00,
    "descuentoAplicado": 5.00,
    "estado": "CONFIRMADA",
    "pago": {
      "id": 456,
      "monto": 45.00,
      "metodoPago": "TARJETA",
      "estado": "COMPLETADO"
    },
    "fechaCreacion": "2025-11-09T10:30:00Z",
    "checkinAt": null,
    "checkoutAt": null,
    "acciones": {
      "puedeModificar": true,
      "puedeCancelar": true,
      "puedeCheckIn": false
    }
  }
}
```

---

### PATCH `/api/v1/reservations/:id`
**Descripción**: Modificar una reserva existente

**Request Body**:
```json
{
  "fechaReserva": "2025-11-16",
  "horaInicio": "14:00",
  "horaFin": "16:00"
}
```

**Response** (200 OK): Reserva modificada

**Errores**:
- `403 FORBIDDEN`: No se puede modificar (menos de 24h para inicio)
- `409 CONFLICT`: Nuevo horario no disponible

---

### DELETE `/api/v1/reservations/:id`
**Descripción**: Cancelar una reserva

**Response** (200 OK):
```json
{
  "success": true,
  "data": {
    "reservaId": 789,
    "estado": "CANCELADA",
    "reembolso": {
      "monto": 45.00,
      "metodoPago": "TARJETA",
      "fechaEstimadaReembolso": "2025-11-12",
      "procesado": true
    }
  },
  "message": "Reserva cancelada. El reembolso se procesará en 2-3 días."
}
```

**Errores**:
- `403 FORBIDDEN`: No se puede cancelar (menos de 24h)

---

### POST `/api/v1/reservations/:id/checkin`
**Descripción**: Realizar check-in digital

**Response** (200 OK):
```json
{
  "success": true,
  "data": {
    "reservaId": 789,
    "checkinAt": "2025-11-15T10:05:00Z",
    "codigoAcceso": "ABC123"
  },
  "message": "Check-in realizado exitosamente"
}
```

---

## 4. Pagos

### GET `/api/v1/payments`
**Descripción**: Historial de pagos del usuario

**Response** (200 OK):
```json
{
  "success": true,
  "data": {
    "pagos": [
      {
        "id": 456,
        "reservaId": 789,
        "monto": 45.00,
        "metodoPago": "TARJETA",
        "estado": "COMPLETADO",
        "fechaPago": "2025-11-09T10:35:00Z",
        "factura": {
          "id": 321,
          "numeroFactura": "WF-2025-00456",
          "urlDescarga": "/api/v1/invoices/321/download"
        }
      }
    ]
  }
}
```

---

### POST `/api/v1/payments`
**Descripción**: Procesar pago para una reserva

**Request Body**:
```json
{
  "reservaId": 789,
  "metodoPago": "TARJETA",
  "stripePaymentMethodId": "pm_xxxxx"
}
```

**Response** (200 OK):
```json
{
  "success": true,
  "data": {
    "pagoId": 456,
    "estado": "COMPLETADO",
    "monto": 45.00,
    "facturaId": 321
  }
}
```

---

## 5. Membresías

### GET `/api/v1/memberships/plans`
**Descripción**: Listar planes de membresía disponibles

**Response** (200 OK):
```json
{
  "success": true,
  "data": {
    "planes": [
      {
        "tipo": "BASICA",
        "nombre": "Básica",
        "precioMensual": 49.00,
        "horasIncluidas": 10,
        "descuento": 0,
        "beneficios": ["10h/mes incluidas", "Acceso a todas las sedes"]
      },
      {
        "tipo": "PROFESIONAL",
        "nombre": "Profesional",
        "precioMensual": 99.00,
        "horasIncluidas": 30,
        "descuento": 15,
        "beneficios": ["30h/mes", "15% descuento adicional", "Prioridad en reservas"]
      }
    ]
  }
}
```

---

### POST `/api/v1/memberships`
**Descripción**: Suscribirse a una membresía

**Request Body**:
```json
{
  "planTipo": "PROFESIONAL",
  "metodoPago": "TARJETA",
  "stripePaymentMethodId": "pm_xxxxx"
}
```

---

## 6. Administración

### GET `/api/v1/admin/dashboard`
**Descripción**: Métricas para dashboard administrativo

**Headers**: `Authorization: Bearer <admin_token>`

**Response** (200 OK):
```json
{
  "success": true,
  "data": {
    "metricas": {
      "reservasHoy": 45,
      "ingresosMes": 12350.00,
      "tasaOcupacionPromedio": 67.5,
      "usuariosActivos": 234
    },
    "tendencias": {
      "reservasPorDia": [12, 15, 18, 22, 19, 24, 27],
      "ingresosPorSemana": [2000, 2500, 2800, 3100]
    },
    "alertas": [
      {
        "tipo": "WARNING",
        "mensaje": "Sala Meeting Room 3 con baja ocupación esta semana"
      }
    ]
  }
}
```

---

### GET `/api/v1/admin/reports/occupancy`
**Descripción**: Generar reporte de ocupación

**Query Parameters**:
- `fechaDesde`, `fechaHasta` (requeridos)
- `sedeId` (opcional)
- `formato` (opcional): JSON | PDF | EXCEL

**Response** (200 OK):
```json
{
  "success": true,
  "data": {
    "reporte": {
      "tipo": "OCUPACION",
      "periodo": "2025-11-01 a 2025-11-30",
      "tasaOcupacionPromedio": 72.3,
      "espaciosMasUsados": [
        {"espacioId": 45, "nombre": "Sala Innovation", "ocupacion": 89.5}
      ],
      "urlDescarga": "/api/v1/reports/123/download"
    }
  }
}
```

---

## Códigos de Estado HTTP Utilizados

| Código | Significado | Uso en WorkFlex |
|--------|-------------|-----------------|
| `200 OK` | Éxito | GET, PATCH exitosos |
| `201 Created` | Recurso creado | POST de reserva, usuario |
| `204 No Content` | Éxito sin contenido | DELETE exitoso |
| `400 Bad Request` | Request inválido | Datos mal formados |
| `401 Unauthorized` | No autenticado | Token inválido/expirado |
| `403 Forbidden` | No autorizado | Sin permisos para acción |
| `404 Not Found` | Recurso no existe | Espacio/reserva no encontrado |
| `409 Conflict` | Conflicto | Espacio ya reservado |
| `422 Unprocessable Entity` | Validación fallida | Email inválido |
| `429 Too Many Requests` | Rate limit excedido | Demasiados requests |
| `500 Internal Server Error` | Error del servidor | Error no manejado |

---

## Versionado de API

### Estrategia
- URL versioning: `/api/v1/`, `/api/v2/`
- Mantener versiones anteriores al menos 6 meses
- Header `X-API-Version: 1.0.0` en respuestas

### Deprecación
```json
{
  "success": true,
  "data": { /* ... */ },
  "warnings": [
    {
      "code": "DEPRECATION_WARNING",
      "message": "Este endpoint será deprecado el 2026-01-01. Migrar a /api/v2/spaces"
    }
  ]
}
```

---

## Documentación API

### Herramientas
- **Swagger/OpenAPI 3.0**: Documentación interactiva
- **Postman Collection**: Para testing manual
- **Auto-generación**: Desde decoradores TypeScript

### URL Documentación
- Desarrollo: `http://localhost:3000/api-docs`
- Producción: `https://api.workflex.com/docs`

---

## Resumen de Endpoints

| Categoría | Endpoints | Métodos Soportados |
|-----------|-----------|-------------------|
| **Auth** | /auth/register, /auth/login, /auth/refresh | POST |
| **Usuarios** | /users/me | GET, PATCH |
| **Espacios** | /spaces, /spaces/:id, /spaces/:id/availability | GET |
| **Reservas** | /reservations, /reservations/:id, /reservations/:id/checkin | GET, POST, PATCH, DELETE |
| **Pagos** | /payments, /payments/:id | GET, POST |
| **Membresías** | /memberships, /memberships/plans | GET, POST, PATCH |
| **Admin** | /admin/dashboard, /admin/reports/* | GET, POST |

**Total**: ~25 endpoints principales en v1
