# Especificación: App de Recepción de Hormigón

## 1. Para quién es y qué problema resuelve

**Usuario:** Encargado de obra que recibe camiones de hormigón en obra.

**Problema:** Hoy el encargado apunta a mano en papel los datos de cada albarán (número, tipo de hormigón, horas, m³, cono) y luego tiene que comprobar manualmente si el cono medido es correcto y si el camión llega dentro del tiempo máximo permitido. Esto es propenso a errores y no genera un registro digital exportable.

**Solución:** Una página web local (sin servidor, sin cobertura) que guía al encargado albarán a albarán, aplica las reglas de aceptación automáticamente y guarda un historial exportable a CSV.

---

## 2. Datos de entrada

### 2a. Una vez por jornada (Pedido del día)
| Campo | Ejemplo |
|---|---|
| Tipo de hormigón pedido | HA-25/B/20/IIa |
| Cono de Abrams pedido (cm) | 9 |

### 2b. Por cada albarán (camión)
| Campo | Ejemplo |
|---|---|
| Número de albarán | 00234 |
| Hormigón indicado en albarán | HA-25/B/20/IIa |
| Hora de salida de planta | 10:15 |
| Hora de llegada a obra | 10:58 |
| Metros cúbicos (m³) | 7,5 |
| Cono de Abrams medido (cm) | 8 |
| Temperatura ambiente (°C) | 22 |

---

## 3. Reglas de decisión

### Regla 1 — Tiempo máximo de transporte
- **Límite fijo:** 90 minutos desde salida de planta hasta llegada a obra.
- **ACEPTAR** si `(hora llegada − hora salida) ≤ 90 min`
- **RECHAZAR** si `(hora llegada − hora salida) > 90 min`
  - Motivo: *"Tiempo de transporte excedido (X min > 90 min)"*

### Regla 2 — Cono de Abrams
- **Tolerancia:** ±2 cm respecto al cono pedido.
- **ACEPTAR** si `|cono medido − cono pedido| ≤ 2 cm`
- **RECHAZAR** si `|cono medido − cono pedido| > 2 cm`
  - Motivo: *"Cono fuera de tolerancia (medido: X cm, pedido: Y cm, diferencia: Z cm)"*

### Regla 3 — Tipo de hormigón
- La comparación ignora mayúsculas/minúsculas y espacios (`ha-25/b/20/iia` = `HA-25/B/20/IIa`).
- **ACEPTAR** si el hormigón del albarán coincide con el pedido (tras normalizar).
- **RECHAZAR** si hay discrepancia.
  - Motivo: *"Tipo de hormigón no coincide (albarán: X, pedido: Y)"*

### Regla 4 — Temperatura ambiente
- **Rango válido:** 5 °C ≤ temperatura ≤ 35 °C.
- **ACEPTAR** si `5 ≤ temperatura ≤ 35`
- **RECHAZAR** si `temperatura < 5` o `temperatura > 35`
  - Motivo: *"Temperatura fuera de rango (X °C)"*

> Un camión se **RECHAZA** si falla **cualquiera** de las cuatro reglas. Se indica el motivo de cada regla que falla.

---

## 4. Pantallas

| # | Pantalla | Contenido |
|---|---|---|
| 1 | **Pedido del día** | Campos: tipo de hormigón + cono pedido. Botón "Iniciar jornada". |
| 2 | **Registro de albarán** | Campos de entrada 2b. Botón "Evaluar". |
| 3 | **Resultado** | Muestra ACEPTADO ✅ o RECHAZADO ❌ con motivo(s). Resumen del albarán. Botón "Siguiente albarán". |
| 4 | **Historial de la jornada** | Lista de todos los albaranes registrados con su resultado. Botón "Exportar CSV". Botón "Nueva jornada". |

---

## 5. Qué NO hará la app

- No se conectará a ningún servidor ni base de datos externa.
- No enviará ni recibirá datos por red.
- No gestionará usuarios, contraseñas ni perfiles.
- No imprimirá albaranes ni generará PDFs.
- No calculará dosificaciones ni resistencias del hormigón.
- No validará el formato interno del tipo de hormigón (solo normaliza y compara texto).

---

## 6. Almacenamiento del historial

El historial de la jornada se guarda en **`localStorage`** del navegador del móvil. Persiste aunque se cierre la página o se apague la pantalla. Solo se borra cuando el encargado pulsa **"Nueva jornada"**. No se requiere conexión a red en ningún momento.

---

## 7. Casos de prueba

| # | Pedido del día | Albarán | T. salida | T. llegada | Cono medido | Temperatura | Resultado esperado |
|---|---|---|---|---|---|---|---|
| P1 | HA-25/B/20/IIa, cono 9 cm | HA-25/B/20/IIa | 08:00 | 09:00 | 9 cm | 20 °C | ✅ ACEPTADO (60 min, cono OK, tipo OK, temperatura OK) |
| P2 | HA-25/B/20/IIa, cono 9 cm | HA-25/B/20/IIa | 08:00 | 09:31 | 9 cm | 20 °C | ❌ RECHAZADO — Tiempo excedido (91 min > 90 min) |
| P3 | HA-25/B/20/IIa, cono 9 cm | HA-25/B/20/IIa | 10:00 | 10:45 | 6 cm | 20 °C | ❌ RECHAZADO — Cono fuera de tolerancia (medido 6 cm, pedido 9 cm, diferencia 3 cm > 2 cm) |
| P4 | HA-25/B/20/IIa, cono 9 cm | HA-30/B/20/IIa | 10:00 | 10:45 | 9 cm | 20 °C | ❌ RECHAZADO — Tipo de hormigón no coincide (albarán: HA-30/B/20/IIa, pedido: HA-25/B/20/IIa) |
| P5 | HA-25/B/20/IIa, cono 9 cm | HA-25/B/20/IIa | 07:30 | 09:02 | 11 cm | 20 °C | ❌ RECHAZADO — **Solo** por tiempo (92 min > 90 min). Cono: diferencia = 11 − 9 = 2 cm ≤ 2 cm → ACEPTADO. Tipo: coincide → ACEPTADO. Temperatura OK. |
| P6 | HA-25/B/20/IIa, cono 9 cm | **ha-25/b/20/iia** | 09:00 | 09:50 | 10 cm | 25 °C | ✅ ACEPTADO — Tipo OK (normalizado coincide), tiempo 50 min ≤ 90 min, cono diferencia 1 cm ≤ 2 cm, temperatura OK |
| P7 | HA-25/B/20/IIa, cono 9 cm | HA-25/B/20/IIa | 08:00 | 09:00 | 9 cm | **3 °C** | ❌ RECHAZADO — Temperatura fuera de rango (3 °C < 5 °C) |
