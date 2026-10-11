# Especificación técnica de alto nivel — JFXAI4ATS

Baseline propuesta ATS-MBSE-001 · 2026-10-11 · Estado: documentación para revisión.
Fuente de alcance: [README del proyecto](../../README.md). El árbol de main inspeccionado contenía únicamente README.md; no hay evidencia de motor de trading implementado. Los requisitos expresan comportamiento futuro, no resultados ejecutados.

## Propósito, límites y actores

Construir una plataforma modular para investigación cuantitativa, asistencia de IA, simulación, órdenes, riesgo y reconciliación. El primer incremento será un escenario **offline y sintético**, con un instrumento, una cuenta paper y un venue simulado. Live, credenciales de broker, rentabilidad y cumplimiento normativo específico quedan fuera de la aceptación de ese incremento.

Investigación define hipótesis y datos; riesgo define límites y acepta exposición; operaciones gestiona parada y reconciliación; mantenedores revisan contratos y evidencia. El asistente de IA propone especificaciones, código y casos; la decisión de aceptar una baseline pertenece a ingeniería. Responsable nominal, presupuesto, fechas de implementación y jurisdicción: pendientes de planificación.

La descripción histórica del repositorio menciona Odoo, GraalVM y GraalPy. Esta baseline no establece una integración Odoo ni compatibilidad GraalPy. Se deberá evaluar JVM/Python por contrato y registrar versiones/licencias antes de adoptar dependencias. FIX, cloud, BI y modelos RL son extensiones; no son requisitos de instalación del piloto offline.

## Necesidades y escenarios

| Necesidad | Escenario operacional | Requisitos derivados |
| --- | --- | --- |
| SN-01 Investigación reproducible | Repetir señales y resultados con datos versionados | ATS-F-001..004, ATS-N-001 |
| SN-02 Control de exposición | Evaluar intención y denegar ante ausencia de información válida | ATS-F-005..006, ATS-F-011, ATS-N-002 |
| SN-03 Integridad de órdenes y libro | Ack, fill parcial, duplicado, cancelación, timeout y reconciliación | ATS-F-007..010, ATS-N-003..004 |
| SN-04 Evidencia de ingeniería | Reconstruir decisiones y verificar cada requisito | ATS-F-012, ATS-N-005..006 |

SN-01..04 se derivan del alcance documental; necesitan validación con los actores. P0: obligatorio para el piloto aplicable; P1: arquitectura prevista con adopción posterior. Los casos definidos aquí son **planificados**. Su definición no implica que hayan pasado.

## Requisitos verificables

| ID | Nombre | Prioridad | Obligación del sistema | Criterio de aceptación | Responsable / verificación |
| --- | --- | --- | --- | --- | --- |
| ATS-F-001 | Datos canónicos | P0 | El sistema deberá validar esquema, instrumento, moneda, unidades, timestamp UTC, origen y versión antes de entregar datos a estrategias. | Un evento válido produce un registro canónico; un evento malformado o fuera del umbral de frescura configurado queda en cuarentena y no produce señal. | B-DATA / TC-001 |
| ATS-F-002 | Proveniencia y replay | P0 | El sistema deberá identificar cada dataset por versión y hash y conservar la configuración usada por cada ejecución. | Dos replays del mismo dataset, semilla, reloj y configuración producen la misma secuencia de eventos normalizados. | B-DATA / TC-002 |
| ATS-F-003 | Propuestas de IA | P0 | El sistema deberá convertir recomendaciones de IA en propuestas estructuradas con modelo, versión, fuentes, estrategia y correlación. | Una salida sin esquema o proveniencia es rechazada antes de generar una intención; ninguna propuesta concede permisos de ejecución. | B-AI / TC-003 |
| ATS-F-004 | Validación de estrategia | P0 | El sistema deberá evaluar estrategias con backtest, separación temporal, costos, slippage y estrés antes de proponer su promoción. | El informe identifica dataset, ventanas temporales, parámetros y métricas; un resultado fuera de los criterios configurados impide la promoción. | B-SIM / TC-004 |
| ATS-F-005 | Riesgo determinista | P0 | El sistema deberá evaluar cada intención contra límites versionados de posición, nominal, exposición y pérdida, usando una posición reconciliada vigente. | Para un límite nominal sintético de 1000 USD, 999 y 1000 pasan ese control y 1001 se rechaza; una política o posición ausente/vencida provoca denegación. | B-RISK / TC-005 |
| ATS-F-006 | Autorización y entornos | P0 | El sistema deberá separar research, paper y live y exigir autorización explícita vinculada a estrategia, cuenta, entorno y política. | Una aprobación paper no habilita live; una aprobación ausente, expirada o de otra versión impide enviar órdenes. | B-ACCESS / TC-006 |
| ATS-F-007 | Ciclo de orden | P0 | El sistema deberá conservar estados y transiciones de intención, rechazo, envío, aceptación, fill parcial, fill total, cancelación y fallo. | Las transiciones inválidas son rechazadas; un timeout queda en estado desconocido hasta reconciliación y no provoca reenvío ciego. | B-OMS / TC-007 |
| ATS-F-008 | Idempotencia | P0 | El sistema deberá deduplicar intenciones y reportes y recuperar el estado persistido tras un reinicio. | Repetir una intención con igual clave y contenido produce un único envío; igual clave con contenido distinto se rechaza; repetir un fill no cambia dos veces posición o efectivo. | B-OMS / TC-008 |
| ATS-F-009 | Adaptadores | P0 | El sistema deberá aislar la semántica externa FIX/API mediante contratos de órdenes, reportes y capacidades del venue. | Un adaptador simulado normaliza ack, fill, rechazo, cancelación y timeout; una capacidad no soportada se rechaza antes del envío. | B-ADAPTER / TC-009 |
| ATS-F-010 | Reconciliación | P0 | El sistema deberá comparar órdenes, fills, posiciones y efectivo internos con el estado reportado por el venue. | Un mismatch genera alerta con correlación y bloquea nuevas órdenes de la cuenta hasta resolución; fills fuera de orden no duplican contabilización. | B-RECON / TC-010 |
| ATS-F-011 | Parada de emergencia | P0 | El sistema deberá bloquear nuevas órdenes al activar kill switch y registrar el resultado de solicitudes de cancelación. | Tras activar parada, el contador de nuevos envíos queda en cero; cancelaciones fallidas conservan evidencia y exposición pendiente. | B-RISK / TC-011 |
| ATS-F-012 | Auditoría y observabilidad | P0 | El sistema deberá registrar correlación, decisión, versión de política/modelo, actor, timestamps y cambios de estado sin secretos. | Una intención permite reconstruir su recorrido completo y causa de rechazo; una prueba con secreto sintético no lo encuentra en logs ni informes. | B-AUDIT / TC-012 |
| ATS-N-001 | Reproducibilidad | P0 | El sistema deberá producir un manifiesto de ejecución con versiones, hashes, semilla, reloj y resultados esperados. | El piloto offline repite tres veces un escenario con resultados idénticos para estados, posición y efectivo con aritmética decimal. | B-SIM / TC-013 |
| ATS-N-002 | Protección de acceso | P0 | El sistema deberá aplicar privilegio mínimo, secretos externos al código y separación entre proponer, aprobar y operar. | Un rol de investigación no cambia límites ni envía órdenes live; la denegación queda auditada. | B-ACCESS / TC-014 |
| ATS-N-003 | Recuperación segura | P0 | El sistema deberá recuperar órdenes y correlaciones durables después de un fallo y reconciliar antes de reactivar envíos. | Un fallo después de persistir intención y antes/después de ack no genera un segundo envío ni doble fill; el estado incierto mantiene bloqueo. | B-OMS / TC-015 |
| ATS-N-004 | Contratos versionados | P1 | El sistema deberá versionar esquemas y rechazar versiones incompatibles antes de procesar mensajes. | La versión v1 se acepta; una versión mayor desconocida produce rechazo explícito sin efecto en posición. | B-ADAPTER / TC-016 |
| ATS-N-005 | Rendimiento medible | P1 | El sistema deberá medir latencia por etapa, throughput y errores con entorno y carga documentados. | El informe del piloto incluye p50/p95/p99 y conteo de eventos; el objetivo de producción queda pendiente de un presupuesto aprobado. | B-AUDIT / TC-017 |
| ATS-N-006 | Trazabilidad de ingeniería | P0 | El proyecto deberá vincular cada requisito con un componente responsable, contrato cuando aplique y caso de verificación versionado. | Los 18 requisitos tienen relaciones satisfy y verify en el modelo; toda modificación de requisito exige revisión de sus vínculos y evidencia. | B-ENGINEERING / TC-018 |

Los límites monetarios anteriores son fixtures de prueba, no recomendaciones de inversión. Un permiso requiere que **todos** los controles aplicables pasen; igualdad al límite pasa solo el control nominal y puede ser denegada por otros controles.

## Descomposición y asignación

| Bloque SysML | Responsabilidad | Frontera |
| --- | --- | --- |
| B-PLATFORM | JFXAI4ATS, composición de bloques | Sistema de interés |
| B-DATA | Ingesta, normalización, datasets y replay | Entrada externa no confiable |
| B-AI | Modelos, agentes, estrategia e hipótesis versionadas | Propuesta sin privilegio de ejecución |
| B-SIM | Backtest, paper, costos, manifiestos y métricas | Research / simulación |
| B-RISK | Política determinista, exposición y kill switch | Antes de cualquier envío |
| B-ACCESS | Autenticación, roles y aprobación por entorno | Separación research / paper / live |
| B-OMS | Estado de órdenes, persistencia e idempotencia | Único generador de órdenes autorizadas |
| B-ADAPTER | Traducción FIX/API y capacidades | Conexión al venue externo o simulado |
| B-RECON | Libro de fills, posiciones, efectivo y comparación | Estado reconciliado usado por riesgo |
| B-AUDIT | Registro, correlación, métricas y alertas | Todos los eventos relevantes |
| B-ENGINEERING | Gestión de requisitos, configuración y evidencia | Ciclo de desarrollo; fuera del runtime |

## Contratos de interfaz v1 (propuestos)

Común: schema_version, event_id, correlation_id, occurred_at UTC, source y environment. Valores financieros como strings decimales, con moneda e instrumento explícitos; nunca usar float binario como valor contable. Cantidad y precio positivos; side determina signo del efecto. Las claves son opacas y no contienen secretos.

| Contrato | Productor → consumidor | Campos obligatorios / invariantes |
| --- | --- | --- |
| IF-01 MarketEvent | B-DATA → B-AI/B-SIM | instrument_id, event_time, received_at, price, quantity, currency, price_unit, quantity_unit, dataset_version, source_hash; frescura relativa al reloj del escenario |
| IF-02 StrategyProposal | B-AI → B-SIM/B-RISK | strategy_id/version, model_id/version (o baseline determinista), dataset_version, source_refs, signal, rationale; esquema inválido no produce intención |
| IF-03 OrderIntent | B-AI/B-SIM → B-RISK | intent_id, idempotency_key, account_id, strategy_version, instrument_id, side, order_type, quantity, limit_price si aplica, currency, environment; immutable después de validar |
| IF-04 RiskDecision | B-RISK/B-ACCESS → B-OMS | intent_id, allowed, reasons, policy_version, position_snapshot_id, approval_id, evaluated_at, expires_at; OMS rechaza decisión vencida o con intención/entorno distinto |
| IF-05 ExecutionReport | B-ADAPTER → B-OMS/B-RECON | order_id, venue_order_id si disponible, report_id, execution_id para fill, status, cumulative_quantity, last_quantity, fill_price, fee y moneda; deduplicar por venue/cuenta/execution_id |
| IF-06 PositionSnapshot | B-RECON → B-RISK | account_id, snapshot_id, as_of, instrument_id, quantity, cash_by_currency, reconciliation_status; mismatch bloquea permisos |
| IF-07 AuditEvent | Todos → B-AUDIT | actor, action, entity_id, versions, outcome, reason, occurred_at, correlation_id; sin credenciales |
| IF-08 EmergencyCommand | Operaciones → B-RISK/B-OMS | actor, account_id, action, reason, command_id, occurred_at; activación bloquea nuevos envíos; cancelación externa requiere confirmación por reporte |

Un timeout de envío es ambiguo: persistir estado Unknown, consultar/reconciliar con el venue y mantener bloqueo. Un retry solo puede ocurrir tras demostrar que no se envió la orden y respetar la semántica idempotente del adaptador. No se promete exactly-once a través de redes externas.

## Caso de aceptación del primer piloto

Fixture mínimo: cuenta paper SYN-ACCOUNT-01 con 10000 USD, instrumento sintético SYN-USD, cotización 100 USD, sin fees, límite nominal 1000 USD y límite posición 10 unidades. Precio y moneda constantes para comprobar el libro; parámetros versionados.

1. BUY 2: permitido con política y aprobación vigentes; dos fills de 1 a 100 producen posición 2 y efectivo 9800.
2. Repetir la intención y el segundo fill: un único envío y el mismo saldo/posición.
3. BUY 11 a 100: rechazo nominal 1100 > 1000; libro sin cambios.
4. Dato viejo, política vencida, aprobación de otra cuenta, salida IA inválida o kill switch activo: cero nuevos envíos y causa auditada.
5. Reporte externo con posición 3 frente a 2: mismatch, alerta y bloqueo de la cuenta.
6. Caída/timeout alrededor de ack: recuperación sin envío duplicado y bloqueo hasta resolver estado.

TC-001..018 describen verificación del sistema completo; el primer piloto ejecutará las partes offline de TC-001..003, TC-005..012, TC-013..016 y trazabilidad TC-018. TC-004 (backtest estadístico completo), TC-017 bajo carga real y toda integración externa seguirán pendientes.

Evidencia requerida: fixtures versionados, manifiesto, resultados esperado/observado, comandos de reproducción, informe de fallos y matriz requisito→caso→resultado. La IA puede producir un primer borrador; ingeniería revisa oráculos, causalidad y regresiones antes de aceptar.

## Decisiones abiertas y riesgos

| ID | Decisión / riesgo | Tratamiento y condición de cierre |
| --- | --- | --- |
| OD-01 | Broker, activos y jurisdicción de live | Bloquea live; elegir con revisión de integración y obligaciones aplicables |
| OD-02 | Latencia, carga, disponibilidad, RTO/RPO | Medir piloto; acordar objetivos numéricos antes de validación de producción |
| OD-03 | Umbrales de frescura y límites | Configuración explícita en fixture; valores live aprobados por riesgo |
| OD-04 | JavaFX/JVM, GraalPy, Python y dependencias | Spike de compatibilidad, versiones y licencias; no asumir bibliotecas compatibles |
| OD-05 | Persistencia, retención y protección de auditoría | Piloto local durable; diseño operativo requiere política aprobada |
| OD-06 | Métricas de promoción y costos de mercado | Definir por estrategia; resultados históricos no equivalen a aceptación live |

Ninguna decisión abierta bloquea escribir la arquitectura; OD-03 y elección de almacenamiento offline deberán resolverse para aceptar el piloto. Ningún requisito queda verificado solo por tener un diagrama.

## Modelo y trazabilidad

[Modelo nativo draw.io](../../MBSE/SysML/jfxai4ats-high-level.drawio): seis páginas editables con elementos y conectores, no imágenes incrustadas. Usa notación gráfica SysML 1.x (req, bdd, ibd, act); no es un repositorio semántico/XMI ni un modelo SysML v2 ejecutable.

- Context: actores, sistema de interés y sistemas externos.
- Requirements: SN→requirement mediante deriveReqt y jerarquía de necesidades.
- Blocks: composición de B-PLATFORM y asignación de responsabilidades.
- Internal interfaces: partes y flujos IF-01..08.
- Activity: validación, aprobación, riesgo, rechazo, envío y reconciliación.
- Verification: dependencias satisfy (bloque→requisito) y verify (testCase→requisito), ambas con origen en quien satisface/verifica.

La matriz normativa es la tabla de requisitos de esta especificación; los IDs deberán permanecer sincronizados con el modelo y con [desarrollo iterativo](../development/iterative-development.md).

