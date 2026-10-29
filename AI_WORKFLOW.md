# AI Workflow & Tooling Report (`AI_WORKFLOW.md`)

Este documento detalla el uso de herramientas de Inteligencia Artificial como parte del proceso de desarrollo, depuración y resolución de la prueba técnica de ingeniería de datos para Twin Hearth Studios.

---

## 1. Herramientas de IA Utilizadas y Propósito

* **Gemini (Asistente Colaborativo Personal):** Utilizado como arquitecto de código principal, guía de resolución de problemas, depurador de consultas SQL complejas (SQLite) y estructurador de documentación técnica.
* **Entorno Local (VS Code + Terminal):** Utilizado para la ejecución iterativa de los scripts de construcción (`bin/build.py`), las pruebas de regresión (`bin/test.py`) y la validación de resultados en el almacén de datos SQLite (`build/warehouse.db`).

---

## 2. Prompts Concretos y Resultados Obtenidos

1. **Prompt 1 (Modelado de Precios y FX):**
   > *"El modelo `03_stg_order_priced.sql` no maneja correctamente las tasas de cambio en días de fin de semana debido a un JOIN directo. ¿Cómo puedo refactorizarlo usando una subconsulta correlacionada basada en fechas?"*
   * **Resultado:** Se obtuvo la estructura de subconsulta con `fx.rate_date <= s.order_date ORDER BY fx.rate_date DESC LIMIT 1`, asegurando que cada transacción obtenga la tasa de cambio válida más reciente sin perder registros de fines de semana.

2. **Prompt 2 (Lógica de Reembolsos y Grano):**
   > *"Necesito actualizar `04_fact_order.sql` para integrar la tabla `raw_refunds` (parciales y totales), mantener el grano exacto a nivel de orden y excluir cuentas de prueba/QA de `raw_players`."*
   * **Resultado:** Se generó la arquitectura con múltiples CTEs (`base_orders`, `refund_aggregation`, `order_items_summary`) que resolvieron el problema de la multiplicación artificial de filas y aplicaron las reglas de negocio de filtrado de calidad de datos.

3. **Prompt 3 (Script de Análisis Financiero Q2):**
   > *"Escribe un script en Python para consultar `warehouse.db`, mapear el trimestre fiscal correcto para FY2026 Q2 (detectado en `dim_date` como `fiscal_quarter_no = -2`) y desglosar el ingreso neto por título y el total global."*
   * **Resultado:** Se creó y ejecutó el script `bin/calc_q2.py`, el cual confirmó la cifra oficial exacta de **$102,609.12** y el desglose por videojuego.

---

## 3. Momento de Alucinación / Corrección (IA Confiadamente Equivocada)

* **El Problema:** Inicialmente, al consultar los ingresos netos de Q2, la IA sugirió filtrar la dimensión de fechas utilizando el estándar convencional (`fiscal_quarter_no = 2` y año fiscal `'FY2026'`). 
* **La Detección:** Al ejecutar el script, la consulta devolvió `$0.00` y arrojó un error de tipo por valores nulos. Mediante la inspección directa de la base de datos con una consulta exploratoria (`SELECT DISTINCT fiscal_year, fiscal_quarter_no FROM dim_date`), se descubrió que el trimestre Q2 en el calendario fiscal personalizado de Twin Hearth Studios estaba mapeado con el identificador numérico `-2` para enero de 2026.
* **La Corrección:** Se ajustó el filtro de la consulta a `d.fiscal_quarter_no = -2`, lo que permitió desbloquear los cálculos financieros correctos y validar la integridad del modelo de fechas.

---

## 4. Estructuración del Flujo de Trabajo

El trabajo se estructuró de forma secuencial y modular para maximizar el rendimiento dentro del límite de tiempo:
1. **Fase 1 (Diagnóstico):** Ejecución inicial de `build.py` y `test.py` para establecer la línea base y analizar la estructura de la base de datos de origen.
2. **Fase 2 (Refactorización del Pipeline):** Corrección iterativa de los modelos SQL de staging (`03`) y fact tables (`04`) resolviendo los defectos de negocio uno a uno.
3. **Fase 3 (Verificación Analítica):** Desarrollo y ejecución del script de Python (`bin/calc_q2.py`) para extraer y auditar los montos definitivos.
4. **Fase 4 (Documentación y Cierre):** Compilación de hallazgos (`FINDINGS.md`) y trazabilidad de IA (`AI_WORKFLOW.md`).