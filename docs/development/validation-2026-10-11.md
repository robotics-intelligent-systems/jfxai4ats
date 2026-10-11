# Validación documental — ATS-MBSE-001

2026-10-11. Estado: documentación propuesta para revisión. No se ejecutaron pruebas del sistema de trading.

| Comprobación | Resultado observado |
| --- | --- |
| Mermaid | 13 diagramas aceptados por el parser Mermaid 11.17.2; flujos con rechazo y control de promoción |
| XML draw.io | Documento XML válido, 6 páginas nativas, IDs únicos por página y referencias de conectores válidas |
| Trazabilidad | 18 requisitos, 18 relaciones satisfy y 18 verify; IDs coinciden con la especificación |
| Texto del modelo | Verificado ajuste de texto con revisión de vistas de contexto, requisitos y matriz; las vistas auxiliares son aproximaciones de layout, no exportaciones del editor draw.io |
| Aceptación funcional | TC-001..018 planificados; ejecución pendiente en ATS-PILOT-002 y posteriores incrementos |

La comprobación de sintaxis Mermaid no mide rendimiento del sistema ni confirma ejecución de órdenes. El XML define notación gráfica SysML 1.x editable; detalle de puertos, despliegue y modelo semántico quedan para ingeniería posterior. Antes de aprobar baseline, un revisor deberá abrir las seis páginas en draw.io y revisar conexiones y asignaciones.

## Reproducir inspección

1. Abrir README.md en un renderizador Mermaid compatible con v11.
2. Abrir MBSE/SysML/jfxai4ats-high-level.drawio en diagrams.net y recorrer Context, Requirements, Blocks, Internal interfaces, Activity y Verification.
3. Comparar ATS-F-001..012 y ATS-N-001..006 con docs/requirements/high-level-specification.md.
4. Confirmar dirección de satisfy/verify hacia el requisito y distinguir casos planificados de resultados ejecutados.
5. Revisar OD-01..06, tabla iterativa y tareas #2 / #1 antes de aceptar.
