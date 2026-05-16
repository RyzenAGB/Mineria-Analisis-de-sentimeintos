# Plan de Análisis de Sentimientos — Reseñas de Celulares en Amazon

## Objetivo

Analizar el sentimiento de reseñas de celulares en Amazon utilizando un enfoque **basado en léxico (VADER)**, enriquecido con vocabulario específico del dominio de telefonía. Comparar los resultados con las calificaciones reales (Rating 1-5) para evaluar la efectividad del método y extraer hallazgos sobre la percepción del consumidor.

---

## Dataset

| Atributo | Detalle |
|---|---|
| **Archivo** | `Amazon-Reviews-Essential.csv` |
| **Tamaño** | ~91 MB |
| **Columnas** | `Rating` (1-5), `Reviews` (texto libre) |
| **Dominio** | Productos de telefonía móvil y accesorios |

---

## Fase 1 — Exploración y Preprocesamiento

### 1.1 Carga y Exploración Inicial
- Cargar el CSV con `pandas`
- Estadísticas descriptivas: total de reseñas, distribución de ratings, longitud promedio de texto
- Detectar y cuantificar valores nulos o vacíos
- Visualizar la distribución de ratings con histograma

### 1.2 Limpieza del Texto
- Convertir a minúsculas
- Eliminar URLs, correos electrónicos y caracteres especiales
- Eliminar números aislados (conservar los que forman parte de modelos como "Galaxy S2")
- Eliminar stopwords en inglés (usando NLTK)
- Aplicar lematización con `WordNetLemmatizer`

### 1.3 Definición de Etiquetas Ground Truth
Convertir el rating numérico a categorías de sentimiento:

| Rating | Sentimiento |
|---|---|
| 1 - 2 | **Negativo** |
| 3 | **Neutro** |
| 4 - 5 | **Positivo** |

---

## Fase 2 — Análisis de Sentimiento con VADER

### 2.1 ¿Por qué VADER?
- Diseñado para texto informal (reseñas, redes sociales)
- Maneja mayúsculas enfáticas ("LOVE this phone!!"), signos de exclamación, negaciones
- Devuelve puntaje `compound` (-1 a +1) que facilita la clasificación
- Librería: `nltk.sentiment.vader`

### 2.2 Enriquecimiento del Léxico con Vocabulario del Dominio

VADER permite agregar palabras personalizadas al léxico. Se creará un **diccionario de dominio** con términos específicos de celulares que VADER no maneja bien por defecto:

| Término | Puntaje | Justificación |
|---|---|---|
| `laggy` | -2.5 | Rendimiento lento, experiencia negativa |
| `lag` | -2.0 | Problema de rendimiento |
| `overheating` | -2.5 | Problema de hardware |
| `overheat` | -2.0 | Problema de hardware |
| `bloatware` | -2.0 | Software preinstalado no deseado |
| `bricked` | -3.5 | Teléfono inutilizable |
| `unresponsive` | -2.5 | Pantalla o sistema que no responde |
| `cracked` | -2.5 | Pantalla rota |
| `refurbished` | -0.5 | Ligeramente negativo (expectativa menor) |
| `glitchy` | -2.0 | Fallos de software |
| `dead pixel` | -2.5 | Defecto de pantalla |
| `battery drain` | -2.5 | Batería que se agota rápido |
| `fast charging` | 2.0 | Característica deseable |
| `snappy` | 2.0 | Rendimiento rápido y fluido |
| `crisp` | 1.5 | Pantalla nítida |
| `flagship` | 1.5 | Gama alta, connotación positiva |
| `waterproof` | 1.5 | Característica deseable |
| `upgrade` | 1.5 | Mejora percibida |
| `downgrade` | -2.0 | Empeoramiento percibido |
| `scratched` | -2.0 | Daño físico |
| `durable` | 1.5 | Resistente, positivo |
| `flimsy` | -2.0 | Materiales de baja calidad |
| `overpriced` | -2.0 | Precio excesivo |
| `bargain` | 2.0 | Buena oferta |
| `scam` | -3.5 | Estafa |
| `knockoff` | -2.5 | Producto imitación |
| `defective` | -3.0 | Producto con defectos |
| `freezing` | -2.0 | Se congela el sistema |
| `smooth` | 1.5 | Funcionamiento fluido |
| `responsive` | 1.5 | Respuesta rápida |

> [!IMPORTANT]
> El enriquecimiento se aplica con `SentimentIntensityAnalyzer().lexicon.update(custom_dict)` antes de ejecutar el análisis.

### 2.3 Clasificación

| Puntaje compound | Sentimiento |
|---|---|
| ≥ 0.05 | Positivo |
| ≤ -0.05 | Negativo |
| entre -0.05 y 0.05 | Neutro |

---

## Fase 3 — Evaluación del Modelo

### 3.1 Métricas de Evaluación
Comparar etiquetas predichas (VADER) vs etiquetas reales (rating):

- **Accuracy** — Porcentaje general de aciertos
- **Precision, Recall, F1-Score** — Por cada clase (Positivo, Negativo, Neutro)
- **Matriz de confusión** — Para identificar dónde falla más el modelo
- **Cohen's Kappa** — Concordancia más allá del azar

### 3.2 Impacto del Enriquecimiento
Ejecutar el análisis **antes y después** de agregar el léxico de dominio para medir si mejora la clasificación. Reportar el delta en accuracy y F1.

---

## Fase 4 — Hallazgos y Análisis Profundo

### 4.1 Análisis de Errores
- **Falsos positivos**: Reseñas con rating bajo clasificadas como positivas (ej: sarcasmo — "Great... it broke in 2 days")
- **Falsos negativos**: Reseñas con rating alto clasificadas como negativas (ej: "no complaints", "never had problems")
- Mostrar **5-10 ejemplos concretos** de cada tipo de error con su texto y puntaje

### 4.2 Análisis de Palabras Clave
- **Nube de palabras** para reseñas positivas vs negativas
- **Top 20 palabras** más frecuentes por sentimiento
- **Bigramas y trigramas** más comunes (ej: "battery life", "customer service", "waste of money")

### 4.3 Análisis Temático
Identificar los temas recurrentes por categoría:

| Sentimiento | Temas esperados |
|---|---|
| Positivo | Duración de batería, calidad de pantalla, buena relación precio-calidad, cámara |
| Negativo | Defectos de hardware, productos usados en mal estado, problemas de carga, estafas |
| Neutro | Reseñas cortas o genéricas, opiniones mixtas |

### 4.4 Relación Longitud del Texto vs Sentimiento
- ¿Las reseñas más largas tienden a ser más negativas o positivas?
- Distribución de longitud por categoría

---

## Fase 5 — Visualizaciones

| # | Visualización | Propósito |
|---|---|---|
| 1 | Histograma de distribución de ratings | Entender el sesgo del dataset |
| 2 | Gráfico de barras: Sentimiento real vs predicho | Comparar distribuciones |
| 3 | Matriz de confusión (heatmap) | Evaluar clasificación |
| 4 | Nube de palabras (positivas / negativas) | Vocabulario dominante |
| 5 | Boxplot de compound score por rating | Relación puntaje léxico ↔ rating |
| 6 | Gráfico de barras: Métricas por clase (P/R/F1) | Desempeño por sentimiento |
| 7 | Gráfico de barras: Top bigramas por sentimiento | Frases más frecuentes |

---

## Fase 6 — Conclusiones

1. **Efectividad del léxico VADER** para clasificar reseñas de celulares
2. **Impacto del enriquecimiento** — ¿Agregar vocabulario de dominio mejoró la clasificación?
3. **Limitaciones** — Sarcasmo, negaciones complejas, reseñas ambiguas
4. **Hallazgos del dominio** — Aspectos que más generan satisfacción e insatisfacción en los consumidores
5. **Recomendaciones** — Posibles mejoras futuras

---

## Stack Tecnológico

| Herramienta | Uso |
|---|---|
| **Python 3.x** | Lenguaje principal |
| **pandas** | Manejo de datos |
| **NLTK** | Tokenización, stopwords, VADER |
| **matplotlib / seaborn** | Visualizaciones |
| **wordcloud** | Nubes de palabras |
| **scikit-learn** | Métricas de evaluación |

---

## Entregables

1. **Notebook Jupyter** con código documentado paso a paso
2. **Visualizaciones** exportadas como imágenes
3. **Reporte de hallazgos** con interpretación y conclusiones

---

> [!TIP]
> Se recomienda trabajar con una **muestra estratificada** (~50,000 reseñas) durante el desarrollo, y ejecutar sobre el dataset completo al final.
