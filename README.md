# Modelo de Ising para Criminalidad en Bogotá

[![Python 3.12](https://img.shields.io/badge/Python-3.12-blue.svg)](https://www.python.org/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)


---

## Resumen

Este proyecto aplica el **modelo de Ising** de la física estadística para modelar la dinámica criminal en Bogotá D.C., utilizando **1,068,340 llamadas** a la línea de emergencias 123 y **4 fuentes de datos abiertos** (estratificación socioeconómica, deserción escolar, Comandos de Atención Inmediata y pobreza).

Cada Unidad de Planeamiento Zonal (UPZ) se trata como un **espín** que puede estar en estado activo (`+1`, alta criminalidad) o inactivo (`-1`, baja criminalidad). La interacción entre UPZs vecinas —modelada mediante la **dinámica de Glauber**— captura el efecto de **contagio espacial** del crimen.

### Hallazgos principales

| Métrica | Valor | Interpretación |
|---------|-------|----------------|
| **AUC** (modelo base) | 0.971 | Predicción casi perfecta de hotspots |
| **J** (contagio) | 0.4336 | Contagio significativo pero moderado |
| **R²** de h_i | 92.5% | Casi toda la variación criminal es explicable |
| **Fase** | Desordenada | T_eff = 1.29 > T_crit = 0.89 |
| **Predictor #1** | Deserción escolar | r = 0.54, p < 0.001 |

---

## Estructura del repositorio

```
proyecto-ising-crimen-bogota/
├── Originales/
│   ├── llamadastramitadas-c4-bogota_...csv
│   └── TAprobacionNOfUPZ_2023.shp
├── Estrato/
│   └── ManzanaEstratificacion.shp
├── Desercion/
│   └── TDesercionOfUPZ.shp
├── CAI/
│   └── ComandoAtencionInmediata.shp
├── pobreza/
│   └── osb_demografia-pobrezaygini.csv
├── graficas/
│   ├── fig1_curva_roc.png
│   ├── fig2_mapa_campos_locales.png
│   ├── fig4_evolucion_R2_correlaciones.png
│   └── fig7_propagacion_criminal.gif
├── modelo_ising_bogota.py
├── graficas_independientes.py
├── presentacion.html
├── informe.pdf
├── referencias.bib
└── README.md
```

---

## Instalación

### Requisitos

- Python 3.10 o superior
- Las siguientes librerías:

```bash
pip install pandas numpy geopandas scikit-learn statsmodels scipy matplotlib
```

### Clonar el repositorio

```bash
git clone https://github.com/usuario/proyecto-ising-crimen-bogota.git
cd proyecto-ising-crimen-bogota
```

### Datos

Los datos crudos **no están incluidos** en este repositorio por su tamaño (>500 MB). Descárgalos de:

- [Llamadas 123 - Datos Abiertos Bogotá](https://datosabiertos.bogota.gov.co/dataset/llamadas-linea-123)
- [Estratificación por manzana](https://datosabiertos.bogota.gov.co/dataset/estratificacion-para-bogota)
- [Deserción escolar por UPZ](https://datosabiertos.bogota.gov.co/dataset/tasa-de-desercion-escolar-en-colegios-oficiales-por-upz-bogota-d-c)
- [CAIs](https://datosabiertos.bogota.gov.co/dataset/centro-de-atencion-accion-para-bogota-d-c)
- [Pobreza y desigualdad](https://datosabiertos.bogota.gov.co/dataset/pobreza-y-desigualdad-en-bogota-d-c)

Coloca los archivos en las carpetas correspondientes según la estructura del repositorio.

---

## Uso

### Ejecutar el modelo completo

```bash
python modelo_ising_bogota.py
```

Esto ejecuta el pipeline completo:
1. Carga y limpieza de datos
2. Construcción de espines y red de vecinos
3. Estimación de los tres modelos (Puro, Base, Sociofísico)
4. Simulación termodinámica (Metropolis)
5. Validación (permutaciones, validación cruzada, ROC)
6. Generación de todas las gráficas

### Generar solo las gráficas

```bash
python graficas_independientes.py
```

Genera 6 figuras PNG (200 DPI) y 1 GIF en la carpeta `graficas/`.

### Ver la presentación interactiva

Abre `presentacion.html` en cualquier navegador. Incluye 22 slides con navegación por teclado, ecuaciones MathJax, gráficas y animación GIF.

---

## Marco teórico

### El modelo de Ising

El modelo de Ising describe sistemas de partículas (espines) con dos estados (`s_i = ±1`) que interactúan con sus vecinos. La energía del sistema es:

```
H = -J Σ s_i s_j - Σ h_i s_i
```

En equilibrio a temperatura T, la probabilidad de una configuración sigue la distribución de Boltzmann:

```
P({s}) = (1/Z) exp(-H / k_B T)
```

### Analogía sociofísica

| Física | Símbolo | Criminología |
|--------|---------|--------------|
| Espín | s_i = ±1 | UPZ con alta/baja actividad criminal |
| Interacción | J | Contagio criminal entre UPZs vecinas |
| Campo externo | h_i | Vulnerabilidad intrínseca de cada UPZ |
| Temperatura | T | Impredictibilidad del sistema |
| Magnetización | M | Actividad criminal global |
| Susceptibilidad | χ | Sensibilidad a intervenciones |

### Dinámica de Glauber (Markoviana)

Para predicción temporal, usamos la dinámica de Glauber (1963):

```
P(s_i(t+1)=+1 | s(t)) = σ(J Σ s_j(t) + h_i)
```

donde σ(x) = 1/(1+e^{-x}) es la función sigmoide.

### Modelo sociofísico enriquecido

Extendemos el modelo para que J y h dependan de covariables observables X_i:

```
J_i = β_0 + Σ β_k X_ik
h_i = α_0 + Σ α_k X_ik
```

---

## Resultados principales

### Comparación de modelos

| Modelo | Ecuación | AUC | Parámetros |
|--------|----------|-----|------------|
| Ising Puro | σ(J · Σ vecinos) | 0.5915 | 1 |
| Ising Base | σ(J · Σ vecinos + h_i) | **0.9710** | 112 |
| Ising Sociofísico | σ(J(X) Σ vecinos + h(X)) | 0.7244 | 15 |

### Coeficientes del modelo sociofísico

| Variable | Efecto sobre J | Efecto sobre h |
|----------|----------------|----------------|
| Deserción escolar | **BLOQUEA** (-0.085) | **AUMENTA** crimen (+0.199) |
| Estrato medio | AMPLIFICA (+0.039) | **REDUCE** crimen (-0.393) |
| N° de CAIs | AMPLIFICA (+0.116) | AUMENTA crimen (+0.088) |

### Fase termodinámica

- T_eff = 1.29 (temperatura efectiva)
- T_crit = 0.89 (temperatura crítica)
- **Fase: DESORDENADA** → hotspots efímeros, se requieren intervenciones sostenidas

### Top 5 UPZs con mayor vulnerabilidad

| # | UPZ | h_i | Deserción | Estrato |
|---|-----|-----|-----------|--------|
| 1 | FONTIBON | +5.21 | 4.4% | 2.8 |
| 2 | VERBENAL | +4.93 | 3.9% | 3.0 |
| 3 | LAS FERIAS | +4.61 | 1.9% | 2.6 |
| 4 | LUCERO | +4.31 | 3.8% | 3.0 |
| 5 | LA SABANA | +4.16 | 3.9% | 3.2 |

---

## Tecnologías utilizadas

| Librería | Versión | Rol |
|----------|---------|-----|
| **Pandas** | 2.2 | Carga CSV, groupby, pivot_table, quantile |
| **NumPy** | 1.26 | Operaciones vectorizadas, álgebra lineal |
| **GeoPandas** | 1.0 | Shapefiles, sjoin, touches, área |
| **Scikit-learn** | 1.5 | LogisticRegression, ROC, TimeSeriesSplit, RF, PCA |
| **Statsmodels** | 0.14 | Logit, OLS, inferencia con p-valores |
| **SciPy** | 1.13 | minimize_scalar, pearsonr |
| **Matplotlib** | 3.9 | Visualización, FuncAnimation, GIF |

---

## Cómo citar

Si usas este trabajo en tu investigación, por favor cítalo como:

```bibtex
@misc{chaparro2025ising,
  author    = {Juan David Chaparro Rojas},
  title     = {Modelo de Ising para Criminalidad en Bogotá: Un Enfoque Sociofísico con Datos Abiertos},
  year      = {2025},
  publisher = {GitHub},
  url       = {https://github.com/usuario/proyecto-ising-crimen-bogota}
}
```

---

## Licencia

Este proyecto está licenciado bajo la **MIT License**.

---

## Contacto

**Juan David Chaparro Rojas**  
Programa Académico de Física  
Universidad Distrital Francisco José de Caldas  
Bogotá D.C., Colombia

---


