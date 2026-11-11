# Detección de Anomalías en Pacientes Ortopédicos mediante Modelos Supervisados de Clasificación

Este repositorio contiene un caso de estudio de clasificación supervisada sobre parámetros biomecánicos de la columna vertebral y la pelvis. El objetivo es construir un pipeline reproducible y libre de fuga de información que permita distinguir pacientes Normales de pacientes Anormales (hernia discal o espondilolistesis), comparando tres modelos (KNN, Random Forest y XGBoost) y priorizando la sensibilidad, porque en este contexto un falso negativo es el error de mayor riesgo clínico.

## Ejecutar en Google Colab

Puedes abrir, revisar y ejecutar todo el código de este análisis de forma interactiva haciendo clic en el siguiente botón:

[![Abrir en Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/KarenTatianaSalazar/deteccion-anomalias-columna-vertebral/blob/main/columna_vertebral_clasificacion.ipynb)

## Resumen del Proyecto

El objetivo principal es evaluar qué modelo ofrece el mejor equilibrio entre exactitud, precisión y capacidad de detección de pacientes anormales, y examinar con qué cuidado debe tratarse una decisión de preprocesamiento que afecta a la variable más importante: los valores negativos del grado de espondilolistesis.

* **Datos utilizados:** base *Vertebral Column* (versión de dos clases) del repositorio de aprendizaje automático de la UCI, con 310 pacientes y seis variables biomecánicas continuas (incidencia pélvica, inclinación pélvica, ángulo de lordosis lumbar, pendiente sacra, radio pélvico y grado de espondilolistesis). La clase Anormal agrupa 210 pacientes (67.7 %) y la clase Normal, 100 (32.3 %). No hay valores nulos. El notebook descarga el archivo directamente desde el repositorio de la UCI.
* **Técnica:** clasificación binaria con KNN, Random Forest y XGBoost, dentro de un pipeline con imputación, escalado, balanceo de clases con SMOTE y reducción de dimensionalidad, y ajuste de hiperparámetros con `GridSearchCV`.
* **Contexto:** desarrollado durante el segundo semestre de la Maestría en Ciencia de Datos de la Universidad Autónoma de Bucaramanga (UNAB).

## Metodología

1. **Análisis exploratorio:** estadísticas descriptivas, asimetría y curtosis, detección de valores atípicos por rango intercuartílico, gráficos por clase y matriz de correlación (que confirma la identidad geométrica PI = PT + SS).
2. **Preprocesamiento:** imputación por mediana, escalado con `StandardScaler`, balanceo con SMOTE aplicado solo sobre el entrenamiento y dentro del pipeline, y reducción de dimensionalidad (sin reducción, `SelectKBest` o PCA) tratada como un hiperparámetro más. La winsorización de atípicos se evaluó con un criterio fijado de antemano y no se aplicó.
3. **Partición y ajuste:** 80 % entrenamiento y 20 % prueba, estratificadas (`random_state = 42`), y búsqueda de hiperparámetros con validación cruzada estratificada de 5 particiones optimizando el F1-Score.
4. **Validación:** validación cruzada repetida (5 particiones × 3 repeticiones) y validación *out-of-bag* del Random Forest.
5. **Evaluación:** métricas en el conjunto de prueba (accuracy, precisión, recall, F1, F2 y ROC AUC), intervalos de confianza de Wilson para el recall, matrices de confusión, curvas ROC e importancia de variables por permutación.
6. **Análisis de sensibilidad:** reentrenamiento bajo cuatro tratamientos de los valores negativos de espondilolistesis (recorte a cero, valores originales, valores originales con indicador de negativo y exclusión de esos registros).

## Principales Resultados

* **Modelo seleccionado:** Random Forest (300 árboles, profundidad máxima de 10, PCA de 4 componentes), elegido con un puntaje ponderado (50 % recall en prueba, 30 % F1 en validación cruzada y 20 % ROC AUC en prueba).
* **Desempeño en prueba (62 pacientes):**

| Modelo | Recall | F1-Score | ROC AUC | Falsos negativos |
|---|---|---|---|---|
| KNN | 0.7143 | 0.7895 | 0.8161 | 12 |
| Random Forest | 0.9286 | 0.8864 | 0.9190 | 3 |
| XGBoost | 0.7381 | 0.8052 | 0.8869 | 11 |

* **Validación cruzada repetida:** Random Forest obtuvo el mayor recall (0.846), F1 (0.862) y ROC AUC (0.882) medios. Las diferencias entre modelos son menores que la variabilidad entre particiones, por lo que no puede afirmarse una superioridad estadísticamente demostrada.
* **Variable más importante:** el grado de espondilolistesis, en los tres modelos.
* **Valores negativos de espondilolistesis:** 58 de los 310 registros (39 % de los Normales y 9 % de los Anormales) se llevaron a cero. Su significado clínico no está documentado en el conjunto de datos y el criterio **no ha sido validado por un especialista**. En el análisis de sensibilidad, al no recortarlos, Random Forest conserva su recall y KNN y XGBoost empeoran levemente; con 62 pacientes de prueba, las diferencias no son concluyentes.
* **Incertidumbre:** el recall de Random Forest tiene un intervalo de Wilson al 95 % de 0.810 a 0.975. Las métricas dependen de una sola partición y de una muestra pequeña.

> El modelo se plantea como apoyo al cribado y no sustituye el criterio del especialista.

## Contenido del Repositorio

* `columna_vertebral_clasificacion.ipynb`: notebook con el análisis completo, desde la introducción hasta las conclusiones y referencias.
* `README.md`: descripción del proyecto.

## Tecnologías

Python, pandas, NumPy, SciPy, scikit-learn, imbalanced-learn, XGBoost, Matplotlib y Seaborn.

## Autora

Karen Tatiana Salazar Sepúlveda — [KarenTatianaSalazar](https://github.com/KarenTatianaSalazar)
