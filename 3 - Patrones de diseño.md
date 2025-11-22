# Patrones de Diseño en WorkFlex

## 1. Patrones Creacionales

### 1.1 Factory Method - Creación de Notificaciones
**Problema**: Diferentes tipos de notificaciones (email, SMS, push) con lógica de creación distinta.

**Solución**:
```
NotificacionFactory
├── crearNotificacionEmail()
├── crearNotificacionSMS()
└── crearNotificacionPush()
```

**Beneficio**: Encapsula la lógica de creación y facilita agregar nuevos tipos sin modificar código existente.

**Ejemplo de uso**:
```python
# Código cliente no necesita saber los detalles de construcción
notificacion = NotificacionFactory.crear(tipo="EMAIL", datos)
notificacion.enviar()
```

---

### 1.2 Builder - Construcción de Reportes Complejos
**Problema**: Los reportes tienen múltiples configuraciones opcionales (fechas, filtros, formatos).

**Solución**:
```
ReporteBuilder
  .setTipo(TipoReporte.OCUPACION)
  .setFechaInicio(fecha1)
  .setFechaFin(fecha2)
  .filtrarPorSede([1, 2, 3])
  .conGraficos(true)
  .formatoExportacion("PDF")
  .build()
```

**Beneficio**: Construcción fluida y legible de objetos complejos con muchos parámetros opcionales.

---

### 1.3 Singleton - Gestor de Configuración
**Problema**: Configuraciones del sistema (URLs de API, credenciales, límites) deben ser únicas y accesibles globalmente.

**Solución**:
```
ConfiguracionManager.getInstance()
  .obtener("PASARELA_PAGO_URL")
  .obtener("MAX_RESERVAS_SIMULTANEAS")
```

**Beneficio**: Punto único de acceso a configuración, inicialización única, evita inconsistencias.

---

## 2. Patrones Estructurales

### 2.1 MVC (Model-View-Controller) - Arquitectura General
**Aplicación**: Patrón arquitectónico principal del sistema.

**Componentes**:
- **Model**: Clases de dominio (Usuario, Reserva, Espacio) + Repositorios
- **View**: Interfaz de usuario (React components, templates)
- **Controller**: Controladores REST que orquestan lógica de negocio

**Flujo**:
```
Usuario → View (UI) → Controller → Service Layer → Model (DB)
                    ← JSON ← JSON ← Datos ←
```

**Beneficio**: Separación clara de responsabilidades, testabilidad, escalabilidad.

---

### 2.2 Repository Pattern - Acceso a Datos
**Problema**: Lógica de negocio no debe conocer detalles de persistencia SQL/NoSQL.

**Solución**:
```
ReservaRepository
├── findById(id)
├── findByCliente(clienteId)
├── findByFecha(fecha)
├── findDisponibles(espacioId, fecha)
└── save(reserva)
```

**Beneficio**: 
- Abstracción del acceso a datos
- Fácil cambio de BD sin afectar lógica de negocio
- Testing con mock repositories

---

### 2.3 Adapter - Integración de Pasarelas de Pago
**Problema**: Diferentes pasarelas (Stripe, PayPal, Redsys) con APIs distintas.

**Solución**:
```
interface PasarelaPago {
  procesarPago(monto, datos)
  verificarEstado(transaccionId)
  reembolsar(transaccionId)
}

StripeAdapter implements PasarelaPago
PayPalAdapter implements PasarelaPago
RedsysAdapter implements PasarelaPago
```

**Beneficio**: Código cliente trabaja con interfaz única, cambio de pasarela sin modificar lógica.

---

### 2.4 Decorator - Cálculo de Precios con Descuentos
**Problema**: El precio de una reserva puede tener múltiples modificadores (membresía, promoción, descuento por volumen).

**Solución**:
```
PrecioBase
├── DescuentoMembresia(wraps PrecioBase)
│   └── DescuentoPromocion(wraps DescuentoMembresia)
│       └── DescuentoVolumen(wraps DescuentoPromocion)
```

**Beneficio**: Añadir/quitar modificadores dinámicamente sin alterar clase base.

---

## 3. Patrones Comportamentales

### 3.1 Strategy - Métodos de Pago
**Problema**: Diferentes algoritmos de procesamiento según método de pago.

**Solución**:
```
interface EstrategiaPago {
  procesar(monto, datos)
}

PagoTarjeta implements EstrategiaPago
PagoPayPal implements EstrategiaPago
PagoTransferencia implements EstrategiaPago

// Uso
procesador.setEstrategia(new PagoTarjeta())
procesador.ejecutar(monto, datos)
```

**Beneficio**: Intercambiar algoritmos en tiempo de ejecución, fácil agregar nuevos métodos.

---

### 3.2 Observer - Sistema de Notificaciones
**Problema**: Múltiples componentes deben reaccionar a eventos (nueva reserva, pago completado, cancelación).

**Solución**:
```
ReservaService (Subject)
├── notificar(evento) →
    ├── EmailNotifier (Observer)
    ├── SMSNotifier (Observer)
    ├── PushNotifier (Observer)
    └── AuditoriaLogger (Observer)
```

**Beneficio**: Desacoplamiento, agregar nuevos observers sin modificar subject.

---

### 3.3 State - Gestión de Estados de Reserva
**Problema**: Una reserva puede estar en múltiples estados con transiciones válidas específicas.

**Solución**:
```
Estados posibles:
PENDIENTE → CONFIRMADA → EN_CURSO → COMPLETADA
         ↓
      CANCELADA

Cada estado define:
- puedeModificar()
- puedeCancelar()
- puedeReembolsar()
```

**Beneficio**: Encapsular comportamiento específico por estado, validaciones claras de transiciones.

---

### 3.4 Command - Operaciones de Reserva
**Problema**: Necesitamos deshacer operaciones (cancelar reserva implica liberar espacio, reembolsar, notificar).

**Solución**:
```
interface Command {
  ejecutar()
  deshacer()
}

CancelarReservaCommand implements Command {
  ejecutar() {
    reserva.cancelar()
    espacio.liberarHorario()
    pago.reembolsar()
    notificacion.enviar()
  }
  deshacer() { /* reversa */ }
}
```

**Beneficio**: Encapsular operaciones complejas, soporte para undo/redo, auditoría.

---

### 3.5 Chain of Responsibility - Validación de Reservas
**Problema**: Múltiples validaciones antes de confirmar reserva (disponibilidad, créditos, límites).

**Solución**:
```
ValidadorDisponibilidad
  → ValidadorCreditos
    → ValidadorLimiteReservas
      → ValidadorHorarioNegocio
        → ConfirmarReserva
```

**Beneficio**: Cada validador maneja su responsabilidad, fácil agregar/quitar validaciones.

---

## 4. Patrones Específicos de API

### 4.1 DTO (Data Transfer Object) - Transferencia de Datos API
**Problema**: No exponer entidades de dominio directamente en API.

**Solución**:
```
ReservaDTO {
  id, espacioNombre, fecha, hora, precio
  // Sin datos sensibles ni relaciones complejas
}

Mapper convierte: Reserva (entidad) ↔ ReservaDTO
```

**Beneficio**: Control sobre qué datos se exponen, versionado de API independiente del modelo.

---

### 4.2 CQRS (Command Query Responsibility Segregation) - Operaciones Complejas
**Problema**: Operaciones de escritura complejas vs consultas optimizadas.

**Solución**:
```
Commands (escritura):
- CrearReservaCommand
- CancelarReservaCommand

Queries (lectura):
- ObtenerReservasQuery
- ObtenerEstadisticasQuery

// Pueden usar diferentes stores/optimizaciones
```

**Beneficio**: Escalabilidad independiente de lecturas/escrituras, optimización específica.

---

## Resumen de Aplicación

| Patrón | Componente | Beneficio Clave |
|--------|-----------|-----------------|
| **Factory** | Notificaciones | Extensibilidad |
| **Builder** | Reportes | Construcción flexible |
| **Singleton** | Configuración | Consistencia global |
| **MVC** | Arquitectura | Separación responsabilidades |
| **Repository** | Datos | Abstracción persistencia |
| **Adapter** | Pagos | Integración múltiples APIs |
| **Decorator** | Precios | Modificadores dinámicos |
| **Strategy** | Métodos pago | Algoritmos intercambiables |
| **Observer** | Eventos | Desacoplamiento |
| **State** | Reserva | Transiciones controladas |
| **Command** | Operaciones | Undo/redo, auditoría |
| **Chain** | Validación | Responsabilidad única |
| **DTO** | API | Seguridad, versionado |
| **CQRS** | Queries/Commands | Escalabilidad |

---

## Principios SOLID Aplicados

### S - Single Responsibility Principle
Cada clase tiene una única razón de cambio:
- `ReservaService`: Lógica de reservas
- `EmailService`: Envío de emails
- `PagoService`: Procesamiento de pagos

### O - Open/Closed Principle
Sistema abierto a extensión, cerrado a modificación:
- Nuevos métodos de pago via Strategy sin modificar código existente
- Nuevas validaciones via Chain of Responsibility

### L - Liskov Substitution Principle
Subclases reemplazables por clase base:
- Cualquier `PasarelaPago` puede sustituir a otra
- `Cliente` y `Administrador` pueden sustituir a `Usuario` en contextos generales

### I - Interface Segregation Principle
Interfaces específicas en lugar de generales:
- `Pagable` vs `Notificable` vs `Reportable`
- Clientes no dependen de métodos que no usan

### D - Dependency Inversion Principle
Depender de abstracciones, no implementaciones:
- Services dependen de `Repository` interface, no implementación SQL específica
- Controllers dependen de `PasarelaPago` interface, no Stripe/PayPal directamente
