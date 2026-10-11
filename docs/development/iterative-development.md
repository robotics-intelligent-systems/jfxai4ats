# Desarrollo iterativo asistido por IA

Fecha: 2026-10-11. [Proyecto 24](https://github.com/orgs/robotics-intelligent-systems/projects/24). La documentación generada necesita revisión antes de aceptar la baseline. No existe evidencia de piloto ejecutado.

| ID | Objetivo / tarea | Entregables | Estado | Aceptación y dependencia | Siguiente acción |
| --- | --- | --- | --- | --- | --- |
| ATS-MBSE-001 / [#2](https://github.com/robotics-intelligent-systems/jfxai4ats/issues/2) | Optimizar flujos y definir requisitos y SysML de alto nivel | 13 Mermaid; 18 requisitos; 8 contratos; draw.io de 6 vistas; informe de validación | In review al vincular PR | Sintaxis Mermaid, XML e IDs comprobados; revisión humana pendiente | Revisar decisiones OD-01..06 y aprobar alcance del piloto |
| ATS-PILOT-002 / [#1](https://github.com/robotics-intelligent-systems/jfxai4ats/issues/1) | Implementar contrato de trades y piloto offline de riesgo y reconciliación | Esquemas v1; fixtures; motor mínimo; pruebas; manifiesto y matriz esperado/observado | Backlog / propuesto | Baseline revisada; almacenamiento local y frescura elegidos; repetir BUY 2 y duplicados sin doble envío/libro; rechazos y fallos bloquean cuenta | Diseñar, implementar y verificar una única cuenta e instrumento sintéticos |

## Secuencia y control

Backlog → Ready → In progress → In review → Done. No se asignan fechas, iteración calendaria, responsables ni esfuerzo sin planificación. In review exige enlace a entregable y evidencia; Done exige revisión. La siguiente tarea amplía #1 y conserva su objetivo original, sin crear una tarea duplicada.

1. Ingeniería revisa requisitos, fronteras, oráculos y decisiones abiertas.
2. IA asiste en contratos, implementación, fixtures, pruebas y documentación.
3. Verificación ejecuta las partes offline de los casos TC definidos en la especificación y guarda versiones, hashes y resultados.
4. Ingeniería acepta o solicita cambios; actualizar perfil con evidencia y retrospectiva.
5. Seleccionar una siguiente integración acotada solo tras evaluar el piloto; live sigue pendiente.

La madurez editorial permanece en arquitectura (~20%). Terminar documentación no equivale a completar el MVP. [Requisitos](../requirements/high-level-specification.md) · [Modelo](../../MBSE/SysML/jfxai4ats-high-level.drawio).
