# Ablaciones del sistema de 4 agentes (v6), Qwen3-4B

Mismas entradas y mismo criterio de corrección (pauta y precedentes del D2) en todas las filas.
Corrección propuesta por Claude y revisable por el equipo; las filas límite están marcadas REVISAR en cada xlsx.

| Configuración | Congeladas (50) | Difíciles (25) | Llamadas al modelo por pregunta | Segundos por pregunta (congeladas / difíciles) |
|---|---|---|---|---|
| Prompting directo | 1 (2 %) | 0 (0 %) | 1 | — |
| RAG D2 | 40 (80 %) | 16 (64 %) | 1 | — |
| **Agentes v6 (completo)** | **46 (92 %)** | **20 (80 %)** | 10,6 | ~110 (todo el set) |
| sin few-shot (Enrutador) | 44 (88 %) | 19 (76 %) | 11,2 / 12,7 | 99 / 145 |
| sin ReAct (Recuperador) | 45 (90 %) | 18 (72 %) | 4,9 / 5,2 | 69 / 99 |
| sin CoT (Redactor) | 46 (92 %) | 19 (76 %) | 9,0 / 10,0 | 51 / 64 |
| sin Verificador | 44 (88 %) | 19 (76 %) | (de las trazas de v6) | — |

## Lectura
- El sistema completo es el mejor o empata en ambos sets. Ninguna ablación individual baja más de 2 preguntas en un set;
  con n = 25 y n = 50, esas diferencias no son estadísticamente significativas.
- Las técnicas son en parte redundantes: el Verificador corrige errores del Redactor, y la herramienta `existe`
  cubre premisas falsas aunque el Enrutador no las detecte.
- Sin few-shot, el Enrutador clasifica 6 preguntas como premisa_falsa (v6: 14) y nunca usa el tipo `cruce`.
- Sin ReAct, la peor caída está en las difíciles (−2): sin búsquedas adicionales falta evidencia (H05 dice que no puede
  calcular; H08 cita el artículo equivocado). Hace menos de la mitad de las llamadas.
- Sin CoT, el tiempo baja a la mitad; pierde preguntas de cálculo y de premisa falsa en las difíciles (H06, H21)
  y gana otras (H07, H14).
