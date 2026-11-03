# Informe técnico: diagnóstico de calidad de datos, La Liga

**Autor:** Kevin
**Actividad:** GA1-220501094-01-AA1-EV06 – Diagnosticar la calidad de datos: preparación, exploración y documentación
**Notebook asociado:** `notebooks/diagnostico_calidad.ipynb`

---

## 1. Nombre del proyecto y pregunta analítica

- **Proyecto:** La Liga (análisis histórico de la Primera División española).
- **Pregunta analítica:** ¿Qué relación tienen los goles a favor, los goles en contra y el rendimiento como local y como visitante con los puntos por partido y la posición final de los equipos en cada temporada, y qué equipos han dominado históricamente La Liga y en qué épocas?

**Nota sobre la pregunta:** la formulación inicial era «¿Qué factores influyen más en la posición final de los equipos?». Tras explorar el dataset se ajustó, porque el archivo solo contiene variables de rendimiento deportivo (goles, puntos, equipos, temporada y jornada) y no incluye otros posibles factores. Por eso los resultados del proyecto se interpretarán como **asociaciones** y no como relaciones causales.

Antes de responder la pregunta es necesario saber si los datos disponibles tienen la calidad y la estructura adecuadas. Este informe documenta ese diagnóstico inicial.

## 2. Descripción de los datasets utilizados

El proyecto utiliza **un único dataset**: `data/raw/la_liga.csv`, cargado en el DataFrame `df_liga`.

Cada fila representa **un partido** de la Primera División entre las temporadas 1929 y 2023. Variables:

| Variable | Descripción |
|---|---|
| `Season` | Temporada (año numérico) |
| `Game` | Número de jornada |
| `Score` | Marcador en texto (por ejemplo `2-3`) |
| `Teams` | Enfrentamiento en texto (`Local - Visitante`) |
| `Home Team` / `Away Team` | Equipo local y equipo visitante |
| `Home Goals` / `Away Goals` | Goles del local y del visitante |
| `Home Points` / `Away Points` | Puntos obtenidos por cada equipo en el partido |
| `Pools` | Resultado de quiniela: `1` gana local, `X` empate, `2` gana visitante |

**Relación con el problema:** los goles, los puntos y la condición de local o visitante son la materia prima para construir la clasificación de cada temporada y analizar cómo se asocian con los puntos por partido y la posición final. La combinación de `Home Team`, `Away Team`, `Season` y los puntos permite además identificar qué equipos han dominado y en qué épocas.

**Archivo:** se encuentra codificado en **UTF-16** (no UTF-8), por lo que se carga con `pd.read_csv(..., encoding="utf-16")`.

## 3. Resultados de la inspección estructural

Se ejecutaron `head()`, `tail()`, `shape`, `columns`, `dtypes`, `info()`, `describe()` y `describe(include="str")`. En la versión de pandas utilizada, las columnas de texto se muestran con el tipo `str`.

### Tabla de diagnóstico estructural

| Dataset | Filas | Columnas | Tipos de datos | Observaciones |
|---|---|---|---|---|
| `df_liga` (`la_liga.csv`) | 26.558 | 11 | 6 numéricas (`int64`) y 5 de texto (`str`) | Un registro por partido, de 1929 a 2023. Sin valores nulos. Codificación UTF-16. No incluye la posición final de los equipos. |

### Hallazgos de la inspección

- `Season` va de 1929 a 2023 y `Game` de 1 a 42.
- Los goles del local promedian **1,80** y los del visitante **1,04**, lo que indica una clara **ventaja de local**.
- `Pools` tiene 3 valores únicos; el valor `1` aparece en 14.474 partidos, es decir en cerca del **54,5 %** de los casos.
- El marcador más frecuente es `1-0` (3.112 veces) y hay 79 marcadores distintos.
- `Home Team` y `Away Team` tienen **65 nombres únicos** cada uno.
- `Teams` tiene 2.448 combinaciones únicas.
- El máximo de `Home Goals` es 12 (Athletic 12-1 Barcelona, 1931). Se revisaron los 5 marcadores más abultados y son valores atípicos **plausibles**, no errores de digitación.
- `Home Points` y `Away Points` tienen un máximo de 3.

## 4. Valores faltantes, tipos de datos e inconsistencias

### 4.1 Valores faltantes

No hay valores faltantes: las 11 columnas tienen 26.558 valores no nulos (`isnull().sum()` devuelve 0 en todas). Tampoco hay filas duplicadas ni partidos repetidos (misma temporada, jornada y equipos).

### 4.2 Tipos de datos

Los tipos son coherentes con lo que representan. Observaciones:

- `Pools` es una variable categórica almacenada como texto, lo cual es adecuado.
- `Score` y `Teams` son texto derivable de otras columnas (redundantes).
- `Season` es entera; sirve para ordenar y agrupar por temporada.

### 4.3 Inconsistencias y verificaciones

| Verificación | Resultado |
|---|---|
| `Score` coincide con `Home Goals`-`Away Goals` | 0 inconsistencias |
| `Teams` coincide con `Home Team - Away Team` | 0 inconsistencias |
| `Home Points` y `Away Points` coinciden con el resultado según la época (2 puntos por victoria hasta 1995 y 3 desde 1996) | 0 inconsistencias |
| `Pools` coincide con los goles | **3 inconsistencias** |
| Filas duplicadas / partidos repetidos | 0 / 0 |

**Registros con `Pools` incorrecto** (en los tres gana el local, pero `Pools` dice `2`; los puntos sí están bien asignados):

| Índice | Temporada | Partido | `Pools` registrado | `Pools` esperado |
|---|---|---|---|---|
| 953 | 1941 | Sevilla 10-3 Valencia | 2 | 1 |
| 5031 | 1960 | Real Madrid 11-2 Elche | 2 | 1 |
| 23672 | 2016 | Real Madrid 10-2 Rayo Vallecano | 2 | 1 |

En los tres casos el local marcó 10 o más goles. Es posible que el error se haya originado al comparar los goles como texto (por ejemplo `"10" < "3"`), aunque esto es una hipótesis que no se verificó.

**Temporadas faltantes:** hay 92 temporadas distintas dentro de un rango de 95 años. Faltan **1937, 1938 y 1939**, que coinciden con el periodo de la Guerra Civil Española, en el que, hasta donde se sabe, no se disputó la Liga.

**Sistema de puntos:** la victoria vale 2 puntos hasta la temporada 1995 y 3 puntos desde 1996. Esto, junto con la ausencia de 1937-1939, sugiere que `Season` corresponde al año en que **termina** la temporada (hipótesis consistente con los datos).

**Tamaño de los torneos:** el número de partidos por temporada varía de 90 a 462 (mediana de 306), porque el número de equipos cambió con el tiempo. Las últimas cinco temporadas (2019 a 2023) tienen 380 partidos cada una, por lo que la última temporada está completa.

**Nombres de equipos:** hay 65 nombres, entre los cuales aparecen variantes:

| Variantes | Temporadas | Conclusión |
|---|---|---|
| `Alavés` / `Deportivo Alavés` | 1931-2021 / 2022 | Mismo club con distinto nombre (no se solapan). Unificar. |
| `R. Sociedad` / `Real Sociedad` | 1929-2021 / 2022-2023 | Mismo club con distinto nombre (no se solapan). Unificar. |
| `Almería` / `Almeria AD` | 2008-2023 / 1980-1981 | Por lo que se sabe, son clubes distintos separados por casi tres décadas. No unificar. |
| `Málaga` / `CD Málaga` | 2000-2018 / 1950-1990 | Por lo que se sabe, son clubes distintos. No unificar. |
| `Burgos` / `Real Burgos CF` | 1972-1980 / 1991-1993 | Por lo que se sabe, son clubes distintos. No unificar. |

Tras unificar los dos primeros pares quedarían 63 equipos únicos. Los pares marcados como clubes distintos deben confirmarse con una fuente externa antes de tomar la decisión final.

## 5. Principales hallazgos de calidad por dataset

### `df_liga`

| Dataset | Problema identificado | Evidencia encontrada | Posible impacto | Acción propuesta |
|---|---|---|---|---|
| `df_liga` | Codificación UTF-16 | `UnicodeDecodeError` al leer con UTF-8 (byte `0xff` en posición 0) | La carga falla si no se especifica la codificación | Leer con `encoding="utf-16"` y documentarlo en el README |
| `df_liga` | No existe la posición final de los equipos | `info()` y `columns` muestran solo datos por partido | No se puede responder la pregunta directamente | Construir la clasificación sumando puntos por equipo y temporada |
| `df_liga` | Sistema de puntos distinto según la época | 2 puntos por victoria hasta 1995 y 3 desde 1996 | Los puntos y posiciones no son comparables entre épocas | Recalcular con 3 puntos por victoria o analizar por periodos |
| `df_liga` | Torneos de distinto tamaño | 90 a 462 partidos por temporada | Los puntos totales no son comparables entre temporadas | Normalizar con puntos por partido |
| `df_liga` | Temporadas faltantes | Faltan 1937, 1938 y 1939 | Vacío en la serie histórica | Documentarlo y no imputar datos |
| `df_liga` | Nombres distintos para un mismo club | `Alavés`/`Deportivo Alavés` y `R. Sociedad`/`Real Sociedad` | Un club se contaría como dos | Unificar los nombres |
| `df_liga` | Pares de nombres parecidos que podrían ser clubes distintos | `Almería`/`Almeria AD`, `Málaga`/`CD Málaga`, `Burgos`/`Real Burgos CF` | Unificar por error mezclaría clubes distintos | Verificar con fuente externa antes de decidir |
| `df_liga` | `Pools` inconsistente | 3 registros con victoria local marcados como `2` | Error menor en una variable categórica | Corregir esos registros o recalcular `Pools` desde los goles |
| `df_liga` | Columnas redundantes | `Score` y `Teams` se pueden reconstruir con otras columnas | Duplicidad de información | Considerar eliminarlas en la fase de limpieza |
| `df_liga` | Pocas variables explicativas | Solo goles, puntos, equipos, temporada y jornada | Limita qué factores se pueden estudiar | Plantearlo como limitación y trabajar con goles, diferencia de goles y localía |
| `df_liga` | Valores atípicos en goles | Goleadas de hasta 12-1 | Podrían sesgar promedios | Mantenerlos: son plausibles. Revisarlos solo si afectan el análisis |

## 6. Acciones de limpieza y preparación propuestas

Esta actividad es de diagnóstico; las acciones se proponen para la siguiente etapa.

1. Cargar siempre con `encoding="utf-16"`.
2. Recalcular `Pools` a partir de `Home Goals` y `Away Goals` para corregir los 3 registros.
3. Unificar `Deportivo Alavés` en `Alavés` y `Real Sociedad` en `R. Sociedad` (o a la inversa), y verificar los demás pares dudosos con una fuente externa.
4. Construir una tabla de clasificación por temporada y equipo (partidos, puntos, goles a favor, goles en contra y diferencia de goles) y derivar la **posición final**.
5. Estandarizar los puntos al sistema de 3 por victoria o trabajar con puntos por partido.
6. Documentar la ausencia de 1937 a 1939 sin imputar valores.
7. Valorar la eliminación de `Score` y `Teams` por ser redundantes.

## 7. Conclusiones

El dataset `la_liga.csv` es de **buena calidad general**: está completo, sin nulos, sin duplicados y con tipos de datos coherentes. Los puntos, los marcadores y los enfrentamientos son consistentes entre sí, salvo 3 registros de `Pools`.

Los problemas encontrados son de **preparación más que de corrupción** de datos: la codificación UTF-16, los nombres de equipos con variantes, los cambios del sistema de puntos y del tamaño de los torneos, y las tres temporadas ausentes.

La principal **limitación** frente a la pregunta analítica es que el archivo contiene resultados por partido y no la posición final. Esta debe construirse. Además, al disponer solo de goles, puntos y localía, los factores que se podrán estudiar se limitan a variables de rendimiento deportivo (goles a favor y en contra, diferencia de goles, rendimiento como local y como visitante), y los resultados serán asociaciones, no causas.

Para analizar el **dominio histórico** de los equipos será indispensable normalizar los puntos (por el cambio de 2 a 3 puntos por victoria en 1996 y por el tamaño variable de los torneos), por ejemplo con puntos por partido, y unificar los nombres de los clubes con variantes. La ausencia de 1937 a 1939 deberá tenerse en cuenta al interpretar las épocas.

**Conclusión:** los datos son adecuados para continuar con el proyecto, una vez se apliquen las acciones de preparación propuestas.

---

## Anexo: reproducibilidad

```python
import pandas as pd
df_liga = pd.read_csv("../data/raw/la_liga.csv", encoding="utf-16")
```

Todas las verificaciones de este informe se encuentran ejecutadas en `notebooks/diagnostico_calidad.ipynb`.
