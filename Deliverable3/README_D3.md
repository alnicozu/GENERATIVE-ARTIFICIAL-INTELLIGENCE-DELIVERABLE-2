# Deliverable 3 — Sistema de 4 agentes sobre Qwen3-4B

Reconstruye el RAG del Entregable 2 como un grafo de **4 agentes** (Enrutador, Recuperador ReAct,
Redactor, Verificador) que usan el mismo `Qwen/Qwen3-4B`, con decisiones y bucles reales.
Diseño completo en `ARQUITECTURA_AGENTES.md`; grafo en `grafo_agentes.png` / `.pdf`.

## Estructura
| Ruta | Contenido |
|------|-----------|
| `prompts/` | Prompt de cada agente, versionado y fuera del código; historial en `prompts/CHANGELOG.md` |
| `agentes/tools.py` | Herramientas sobre el corpus del E2: `buscar`, `traer_articulo`, `existe_articulo` |
| `agentes/orquestador.py` | Conecta los 4 agentes, aplica ramas y bucles, `chequear_citas`, y registra las trazas |
| `agentes/demo_mock.py` | Prueba la lógica sin GPU con un modelo simulado determinista (29 comprobaciones) |
| `agentes/test_set_50.csv` | Set de 50 preguntas del E1/E2 (congelado, sin cambios) |
| `agentes/test_set_dificil_25.csv` / `.json` | Set nuevo de 25 preguntas difíciles, con respuesta y fuente verificadas |
| `D3_colab_qwen3_4b.ipynb` | Corre el sistema con Qwen3-4B real en Colab (T4) sobre los 75 preguntas |
| `traza_ejemplo.json` | Traza de ejemplo (mensajes entre agentes) del caso "Verificador rechaza y re-busca" |

## Probar la lógica sin GPU
```bash
cd agentes && python3 demo_mock.py
```
Debe terminar con 29 líneas `OK` (cruce, premisa falsa, rechazo con re-búsqueda, pregunta numérica,
salidas desordenadas y una regresión con la salida real de Qwen3-4B).

## Correr con el modelo real
Abrir `D3_colab_qwen3_4b.ipynb` en Colab con GPU T4 y ejecutar las celdas en orden. Produce
`resultados_agentes_D3.csv` (columna `correcto` vacía, para corrección **manual**) y
`trazas_D3.json`. Guarda tras cada pregunta: si Colab se desconecta, se vuelve a ejecutar y
continúa donde quedó.

Configuración (igual que el E2): Qwen3-4B en 4 bits NF4, `enable_thinking=False`, temperatura 0,7,
top-p 0,8, top-k 20, min-p 0, semilla base 2026 (fijada por llamada para que sea reproducible),
recuperador `multilingual-e5-small`.

## Topes
ReAct ≤ 3 acciones por sub-consulta; ≤ 2 ciclos de corrección del Verificador. Si se agotan, se
entrega el último borrador marcado como **no aprobado**.

## Estado (2026-10-31)
- Hecho: corridas reales v4, v5 y v6 (75 preguntas cada una), RAG del D2 sobre las 25 difíciles, las cuatro ablaciones
  sobre las 75 preguntas, corrección de todas las corridas y documento LaTeX de una página (`documento/`).
- Resultados: `resultados_ablaciones/RESUMEN_ABLACIONES.md` (completo 46/50 y 20/25; RAG D2 40/50 y 16/25; directo 1/50 y 0/25).

## Reproducir lo que muestra el video
1. Abrir `D3_colab_qwen3_4b.ipynb` en Colab con GPU T4 ([enlace directo](https://colab.research.google.com/github/alnicozu/GENERATIVE-ARTIFICIAL-INTELLIGENCE-DELIVERABLE-2/blob/main/Deliverable3/D3_colab_qwen3_4b.ipynb)).
2. Ejecutar los Pasos 0 a 5d. El Paso 2 clona este repositorio (código, prompts y corpus); no hay que subir archivos.
3. El Paso 5d corre en vivo H06 (cálculo), P25 (el caso de falla del E2), H20 (premisa falsa) y H03 (caso de falla del D3).
   Cada paso de cada agente aparece a medida que ocurre, junto con la respuesta guardada del RAG del E2 y la referencia.
4. La última celda del Paso 5d descarga `trazas_video_D3.json`; las trazas completas de las 75 preguntas están en
   `resultados_v6/trazas_D3.json` y las de cada ablación en `resultados_ablaciones/`.

## Ablaciones (una técnica clave quitada a la vez)
En el notebook, Paso 5c: `ABLACION = None` es el sistema completo (v6); las variantes quitan una sola técnica:

| `ABLACION` | Agente | Qué se quita |
|---|---|---|
| `sin_fewshot` | Enrutador | los ejemplos few-shot (bloque `<ejemplos>`) |
| `sin_react` | Recuperador | el ciclo ReAct: queda solo la búsqueda base con la pregunta original |
| `sin_cot` | Redactor | la instrucción de razonar paso a paso en `<pensar>` |
| `sin_verificador` | Verificador | el bucle de revisión: se entrega el primer borrador |

Se aplican en el código (`agentes/orquestador.py`, `ABLACIONES`) sin modificar los archivos de prompts, y fallan si el
bloque a quitar no existe. Con `ABLACION = None` los prompts enviados al modelo son idénticos a los de la corrida v6
(comprobado en las 75 preguntas). `sin_verificador` también puede obtenerse de las trazas de la v6: el primer borrador es
idéntico (mismo prompt y misma semilla). Las ablaciones finales se corrieron sobre las 75 preguntas (`SOLO_DIFICILES = False`).
