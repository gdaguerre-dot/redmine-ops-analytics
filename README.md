# Del ticket al patrón — Análisis operativo de un sistema de gestión pública

**310 peticiones no son ruido: son el mapa de dónde falla un sistema.**

Un trimestre (sep–dic 2025) de tickets funcionales en una plataforma de administración electrónica para el sector educativo, leído como lo que es: una fuente de datos operativos que puede orientar decisiones, no solo una bandeja de entrada.

🔗 **[Ver el case study interactivo](https://gdaguerre-dot.github.io/redmine-ops-analytics/)** · 📓 [Ver el notebook de análisis](https://github.com/gdaguerre-dot/redmine-ops-analytics/blob/main/notebook/redmine_ticket_analysis.ipynb)

---

## Contexto

Como Analista Funcional en un proyecto de administración electrónica orientado a centros educativos, parte del trabajo diario es relevar, clasificar y dar seguimiento a las peticiones que llegan desde usuarios finales hasta que quedan resueltas. Este proyecto toma un recorte real de esa gestión — un trimestre completo — y lo trata como un dataset operativo, con las mismas preguntas que se le harían a cualquier proceso de negocio: ¿cuándo llega el trabajo?, ¿dónde se concentra?, ¿qué tan rápido se resuelve?, ¿qué tipo de esfuerzo predomina?

> **Nota de confidencialidad:** los datos publicados en este repositorio fueron anonimizados y reducidos a campos estructurales (fecha, módulo, estado, prioridad, categoría). Se excluyeron nombres de personas, asuntos y descripciones de tickets, y cualquier referencia al sistema o al organismo público específico. El dataset original completo no se publica.

## Hallazgos principales

| Pregunta | Hallazgo |
|---|---|
| ¿Hay estacionalidad? | Septiembre concentra ~33% del volumen trimestral — coincide con el arranque del curso escolar |
| ¿Dónde se concentra el trabajo? | Un módulo explica ~46% de las peticiones — más del doble que el segundo en volumen |
| ¿Cuánto está resuelto? | 42% en estado *Resuelta* al cierre, con varias etapas de validación intermedias |
| ¿Predomina lo urgente? | No — 83% de las peticiones son prioridad *Normal* |
| ¿Se corrige o se construye? | El trabajo correctivo duplica al evolutivo (59% vs 28%) |
| ¿Qué módulo genera más fricción, no solo más volumen? | Los temas ligados a gestión económica concentran la mayor proporción de sentimiento negativo (hasta 43%), muy por encima del 13% promedio — el mismo módulo que ya lideraba en volumen |

Ver el desarrollo completo de cada hallazgo en el [case study](https://gdaguerre-dot.github.io/redmine-ops-analytics/) o en el [notebook](https://github.com/gdaguerre-dot/redmine-ops-analytics/blob/main/notebook/redmine_ticket_analysis.ipynb).

## Vista previa

<p>
  <img src="https://github.com/gdaguerre-dot/redmine-ops-analytics/raw/main/assets/01_volumen_mensual.png" width="49%" alt="Volumen mensual" />
  <img src="https://github.com/gdaguerre-dot/redmine-ops-analytics/raw/main/assets/02_modulo.png" width="49%" alt="Peticiones por módulo" />
</p>
<p>
  <img src="https://github.com/gdaguerre-dot/redmine-ops-analytics/raw/main/assets/03_estado.png" width="32%" alt="Estado al cierre" />
  <img src="https://github.com/gdaguerre-dot/redmine-ops-analytics/raw/main/assets/04_prioridad.png" width="32%" alt="Prioridad asignada" />
  <img src="https://github.com/gdaguerre-dot/redmine-ops-analytics/raw/main/assets/05_categoria.png" width="32%" alt="Tipo de trabajo" />
</p>
<p>
  <img src="https://github.com/gdaguerre-dot/redmine-ops-analytics/raw/main/assets/06_topicos.png" width="70%" alt="Tópicos y sentimiento" />
</p>

---

## Estructura del repositorio

```
redmine-ops-analytics/
├── README.md                          este archivo
├── data/
│   ├── tickets_anonimizado.csv         dataset público (7 columnas estructurales, 310 filas)
│   └── topicos_sentimiento_resumen.csv  tópicos + sentimiento agregado (fase 2, sin texto original)
├── notebook/
│   ├── redmine_ticket_analysis.ipynb   análisis reproducible (pandas + matplotlib)
│   └── nlp/
│       └── topic_sentiment_analysis.ipynb   BERTopic + sentiment (español) — corre local, sobre texto no publicado
├── site/
│   └── index.html                     case study interactivo (HTML + Chart.js), publicado vía GitHub Pages
└── assets/
    └── *.png                          gráficos exportados, usados en este README
```

## Cómo reproducirlo

```
git clone https://github.com/gdaguerre-dot/redmine-ops-analytics.git
cd redmine-ops-analytics
pip install pandas matplotlib jupyter
jupyter notebook notebook/redmine_ticket_analysis.ipynb
```

El notebook lee `data/tickets_anonimizado.csv` y regenera los cinco gráficos del análisis con las mismas conclusiones que el case study.

## Pipeline NLP

![Pipeline NLP: preprocessing, TF-IDF, BERTopic, sentiment y segmentación](https://github.com/gdaguerre-dot/redmine-ops-analytics/raw/main/assets/nlp_pipeline.svg)

El texto libre de cada ticket pasa por limpieza y normalización, y de ahí se procesa en dos vías complementarias: **TF-IDF**, que identifica qué términos son característicos de cada grupo (no solo frecuentes), y **BERTopic**, que agrupa los tickets en tópicos latentes. Ambas salidas se cruzan con el resultado de **sentiment analysis** y se segmentan por tipo de trabajo (correctivo/evolutivo), sentimiento (negativo/positivo) y módulo — cada segmento se reporta siempre junto a su tamaño de muestra (`n`), para no comparar grupos de 100 tickets con grupos de 15.

**Términos característicos por segmento** (TF-IDF sobre el corpus completo, promediado por segmento — no frecuencia simple):

| Segmento | n | Términos característicos |
|---|---|---|
| Correctivo | 83 | economica, perfil, error, comunicación, asiento, pantalla |
| Evolutivo | 40 | ventanilla, centro, perfil, servicio, prepago, cargos |
| Gestión económica | 143 | economica, asiento, perfil, pantalla, error, asientos |
| Resto de módulos | 167 | centro, pantalla, servicio, fecha, transporte, nueva |

*(Segmentación por tipo de trabajo y por módulo. Falta agregar el cruce por sentimiento —pendiente de mergear con la salida de `topic_sentiment_analysis.ipynb`—, ver [`notebook/nlp/tfidf_segmentado_wordclouds.ipynb`](https://github.com/gdaguerre-dot/redmine-ops-analytics/blob/main/notebook/nlp/tfidf_segmentado_wordclouds.ipynb). La word cloud general sobre todos los tickets se evaluó y se descartó del entregable final: las palabras más frecuentes no necesariamente representan los problemas más importantes. Las word clouds por segmento —abajo— quedan como apoyo visual de esta tabla, no como reemplazo.)*

![Nubes de palabras por segmento, generadas a partir de los pesos TF-IDF](https://github.com/gdaguerre-dot/redmine-ops-analytics/raw/main/assets/wordclouds_por_segmento.png)

## Stack

`Python` · `pandas` · `matplotlib` · `HTML/CSS` · `Chart.js` · `BERTopic` · `pysentimiento` · `TF-IDF` (scikit-learn) · `Redmine` (fuente de datos original)

## Próximos pasos

- [x] Topic modeling (BERTopic) + análisis de sentimiento (español, `pysentimiento`) sobre el texto de las peticiones — ver [`notebook/nlp/topic_sentiment_analysis.ipynb`](https://github.com/gdaguerre-dot/redmine-ops-analytics/blob/main/notebook/nlp/topic_sentiment_analysis.ipynb). Corre sobre el texto original en un entorno local (no incluido en este repo por confidencialidad) y publica únicamente resultados agregados en `data/topicos_sentimiento_resumen.csv`.
- [x] TF-IDF segmentado (correctivo/evolutivo, gestión económica/resto) para identificar vocabulario característico por grupo, reportando siempre el `n` de cada segmento — ver [`notebook/nlp/tfidf_segmentado_wordclouds.ipynb`](https://github.com/gdaguerre-dot/redmine-ops-analytics/blob/main/notebook/nlp/tfidf_segmentado_wordclouds.ipynb).
- [x] Word clouds por segmento (no general) como apoyo visual de la tabla TF-IDF — ver `assets/wordclouds_por_segmento.png`.
- [ ] Sumar el cruce por sentimiento (negativo/positivo-neutro) a la tabla TF-IDF, una vez mergeado con la salida de `topic_sentiment_analysis.ipynb`.

---

**Gerónimo Daguerre** · Analista Funcional · BI & Data Analytics
[LinkedIn](https://linkedin.com/in/gerodaguerre) · <gerodaguerre@gmail.com>
