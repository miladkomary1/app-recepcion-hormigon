# App de Recepción de Hormigón

Herramienta web para que el encargado de obra registre y valide cada camión de hormigón que llega a obra.

## ¿Qué hace?

- Registra los datos de cada albarán: número, tipo de hormigón, horas de salida/llegada, m³ y cono de Abrams medido.
- Aplica automáticamente las reglas de aceptación (tiempo de transporte ≤ 90 min, cono ±2 cm, tipo de hormigón correcto).
- Muestra **ACEPTADO ✅** o **RECHAZADO ❌** con el motivo concreto.
- Guarda el historial en el propio móvil (sin servidor) y permite exportarlo a CSV para Excel.

## Cómo usarla

1. Descarga o copia el archivo `index.html` en el móvil (o ábrelo desde un ordenador).
2. Ábrelo con cualquier navegador (Chrome, Safari…). **No necesita internet.**
3. Al iniciar la jornada, introduce el tipo de hormigón y el cono pedido.
4. Por cada camión que llegue, rellena los datos del albarán y pulsa **Evaluar**.
5. Al terminar, ve al historial y pulsa **Exportar CSV**.

## Aviso

> **Demo del curso «IA aplicada a la construcción».**
> Los datos introducidos son de ejemplo y no tienen validez contractual.
> Comprueba siempre los requisitos específicos de cada obra y normativa aplicable.
