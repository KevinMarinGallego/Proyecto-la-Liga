# Proyecto La Liga: diagnóstico de calidad de datos

Proyecto individual del curso de Ciencia de datos con Python (SENA). Contiene el diagnóstico inicial de calidad del dataset histórico de partidos de la Primera División española.

## Descripción

El proyecto analiza resultados históricos de La Liga (temporadas 1929 a 2023). En esta etapa se carga el dataset con Pandas, se inspecciona su estructura y se documentan los problemas de calidad y las acciones de limpieza propuestas, antes de hacer cualquier análisis o modelado.

## Propósito

Determinar si el dataset tiene las condiciones adecuadas para responder la pregunta analítica del proyecto y qué preparación necesita.

## Pregunta analítica

**¿Qué factores influyen más en la posición final de los equipos?**

## Dataset

| Archivo | Descripción | Filas | Columnas |
|---|---|---|---|
| `data/raw/la_liga.csv` | Un registro por partido de la Primera División, de 1929 a 2023 | 26.558 | 11 |

Variables: `Season`, `Game`, `Score`, `Teams`, `Home Team`, `Away Team`, `Home Goals`, `Away Goals`, `Home Points`, `Away Points`, `Pools`.

**Fuente del dataset:** [completar con el enlace o nombre de la fuente de donde se descargó].

**Importante:** el archivo está codificado en **UTF-16**. Se debe cargar con:

```python
pd.read_csv("../data/raw/la_liga.csv", encoding="utf-16")
```

Si el archivo no se incluye en el repositorio por su tamaño o por restricciones de distribución, descárgalo desde la fuente indicada y colócalo en `data/raw/la_liga.csv`.

## Estructura del repositorio

```
Proyecto Laliga/
├── README.md
├── .gitignore
├── requirements.txt
├── data/
│   └── raw/
│       └── la_liga.csv
├── notebooks/
│   └── diagnostico_calidad.ipynb
└── reports/
    └── informe_diagnostico.md
```

## Cómo reproducir el diagnóstico

Desde PowerShell, en la raíz del proyecto:

1. Crear y activar el entorno virtual:
   ```powershell
   python -m venv .venv
   .\.venv\Scripts\Activate.ps1
   ```
2. Instalar las dependencias:
   ```powershell
   pip install -r requirements.txt
   ```
3. Abrir JupyterLab:
   ```powershell
   jupyter lab
   ```
4. Abrir `notebooks/diagnostico_calidad.ipynb` y ejecutar todas las celdas (**Kernel → Restart Kernel and Run All Cells**).

## Resultados principales

El informe completo está en [`reports/informe_diagnostico.md`](reports/informe_diagnostico.md). Resumen:

- El dataset no tiene valores nulos ni filas duplicadas.
- `Score`, `Teams` y los puntos son consistentes con los goles; solo 3 registros de `Pools` son incorrectos.
- Faltan las temporadas 1937, 1938 y 1939.
- El sistema de puntos cambia en 1996 (de 2 a 3 puntos por victoria) y el tamaño de los torneos varía entre épocas.
- Hay nombres de equipos con variantes (`Alavés`/`Deportivo Alavés`, `R. Sociedad`/`Real Sociedad`).
- No existe la posición final: debe construirse a partir de los puntos por equipo y temporada.

## Autor

Kevin
