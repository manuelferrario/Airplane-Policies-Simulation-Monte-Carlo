# Simulación de políticas de embarque aéreo — Monte Carlo

**Trabajo Práctico 1 · Aplicaciones Computacionales en Negocios · Universidad Torcuato Di Tella · 2026**
Profesor: Nicolás Merener

**Integrantes:** Manuel Ferrario · Zaki Freidenberg · Débora Melamed · Felipe Vilaseca Navajas

---

## Resumen

La aerolínea ficticia *Di Tella Flying Circus* quiere saber **qué política de embarque llena el avión más rápido** y **cuánto dinero está en juego**. Construimos un simulador de eventos discretos, segundo a segundo, de 100 pasajeros subiendo a un avión de un solo pasillo. Comparamos cuatro políticas clásicas con **5.000 corridas de Monte Carlo por escenario** y tradujimos los tiempos a euros con un modelo de costo de demora.

**Resultado principal:** el **método Steffen** es el más rápido en todos los escenarios. Con la mitad de los pasajeros llevando carry-on tarda **15,7 min**, contra **20,5 min** de Back-to-Front, que es la política más usada en la práctica. Además es la política con menos variabilidad. Pasar de Back-to-Front a Steffen ahorra unos **€312 por vuelo** sin vender un asiento menos.

---

## El problema

| Elemento | Valor |
|---|---|
| Filas | 25 |
| Asientos por fila | 4 (`-2`, `-1` \| pasillo \| `1`, `2`) |
| Pasajeros | 100, cada uno con asiento asignado |
| Pasillo | Una sola persona de ancho |
| Carry-on | Cada pasajero lo lleva con probabilidad `p` |

Cada pasajero camina hasta su fila, guarda el carry-on si lo lleva y se sienta. Si hay alguien sentado entre el pasillo y su asiento, ese pasajero tiene que levantarse para dejarlo pasar. Un pasajero avanza a la fila siguiente solo si ese lugar del pasillo estuvo libre durante los 3 segundos anteriores.

El enunciado completo está en [`Trabajo práctico 1 ACN 2026.pdf`](Trabajo%20práctico%201%20ACN%202026.pdf).

---

## Decisiones de modelado

**Reloj:** la simulación avanza de a 1 segundo.

**Estados de cada pasajero:**

```
esperando → caminando → guardando → (interferencia) → sentándose → sentado
```

**Parámetros de tiempo** (tomados de videos de embarque y de experiencia propia):

| Acción | Tiempo |
|---|---:|
| Caminar una fila sin carry-on | 3 s |
| Caminar una fila con carry-on | 6 s |
| Guardar el carry-on | 10 s |
| Sentarse | 4 s |
| Interferencia (un pasajero sentado se levanta y deja pasar) | 8 s |

**Reglas físicas iguales para todas las políticas:** lo único que cambia entre una política y otra es el **orden de entrada** de los pasajeros. Así, cualquier diferencia en el tiempo se debe a la política y no a cambios en la mecánica.

---

## Políticas evaluadas

| Política | Idea |
|---|---|
| **Random** | Los 100 pasajeros entran en orden aleatorio. |
| **Back-to-Front** | Primero las filas del fondo y después las de adelante (orden aleatorio dentro de cada fila). |
| **WILMA** (*Window-Middle-Aisle*) | Primero todas las ventanillas y después todos los pasillos. |
| **Steffen** | Primero ventanillas y después pasillos, de atrás hacia adelante, alternando filas pares e impares y alternando lados. Así los pasajeros que están en el pasillo al mismo tiempo guardan el equipaje en paralelo, sin bloquearse entre sí. Lo adaptamos a la configuración 2+2, que no tiene asiento del medio. |

---

## Resultados — Monte Carlo (N = 5.000 por política y escenario)

### Tiempo esperado de llenado

| Política | p = 0 (nadie con carry-on) | p = 0,5 (escenario base) | p = 1 (todos con carry-on) |
|---|---:|---:|---:|
| **Steffen** | **10,10 min** | **15,73 min** | **16,63 min** |
| WILMA | 11,20 min | 17,81 min | 20,07 min |
| Random | 11,70 min | 18,26 min | 20,64 min |
| Back-to-Front | 12,15 min | 20,47 min | 26,01 min |

### Precisión y variabilidad (escenario base, p = 0,5)

| Política | Media (s) | Desvío estándar (s) | Error estándar (s) | IC 95% (s) |
|---|---:|---:|---:|---:|
| Steffen | 943,61 | 17,09 | 0,24 | [943,13 ; 944,08] |
| WILMA | 1068,47 | 36,38 | 0,51 | [1067,46 ; 1069,48] |
| Random | 1095,88 | 38,31 | 0,54 | [1094,82 ; 1096,94] |
| Back-to-Front | 1228,39 | 40,17 | 0,57 | [1227,28 ; 1229,50] |

El error estándar de todas las estimaciones es **menor a 1 segundo** sobre tiempos de 15 a 20 minutos. Los intervalos de confianza no se superponen, así que el ranking es estadísticamente sólido.

### Conclusiones

1. **Steffen gana en los tres escenarios** y es la política con menos variabilidad: tiene el desvío estándar más bajo y las cajas más angostas en el boxplot.
2. **Back-to-Front es la peor**, y es la que más empeora con el equipaje: pasa de 12,2 a 26,0 min cuando todos llevan carry-on. Al mandar juntos a pasajeros de filas vecinas, genera colas en el pasillo mientras cada uno guarda su valija.
3. **El carry-on es el factor que más pesa.** Casi todas las políticas tardan cerca del doble cuando todos los pasajeros llevan equipaje de mano.
4. Con `p = 0` y `p = 1`, Steffen es **determinístico** (desvío = 0): el orden de entrada es fijo y ya no queda ninguna fuente de azar.

### Convergencia del estimador

Incluimos un gráfico log-log del ancho del IC 95% en función de N. La recta con pendiente −½ confirma que el error cae como **1/√N**, como predice la teoría. Con N = 5.000 el error ya es despreciable para el problema.

---

## Impacto económico

**¿Cuánto cuesta una demora de 30 minutos?** Usamos una tabla de costos de demora en puerta para un **Embraer E190**, un avión de tamaño similar al del problema, e interpolamos linealmente entre sus puntos:

| Demora | Costo |
|---|---:|
| 5 min | €71 |
| 15 min | €380 |
| 30 min | €1.366 |

**Ahorro por elegir bien la política:** con p = 0,5, pasar de Back-to-Front a Steffen ahorra **≈ €312 por vuelo** en costo de demora.

**¿Conviene dejar asientos vacíos para embarcar más rápido?** **No.** Con una tarifa base de €75, la que usamos para un tramo corto de cabotaje, 30 minutos de demora equivalen al ingreso de **~18 asientos**, casi una quinta parte del avión. Sacar 2 o 3 pasajeros acorta el embarque en segundos, no en media hora. Esto coincide con lo que hacen las aerolíneas reales, que practican *overbooking* justamente porque un asiento vacío cuesta más que cualquier demora operativa razonable. **La primera palanca es la política de embarque, no bajar la ocupación.**

---

## Bonus (+30%): heterogeneidad y segmentación por clases

Extendimos el modelo sin modificar la simulación original:

- **Clases y grupos de embarque:** Primera Clase en las filas 1-5, que embarca primero. Después vienen tres grupos económicos en bloques contiguos de filas: Grupo 1 (filas 6-11), Grupo 2 (12-17) y Grupo 3 (18-25).
- **Tipos de pasajero**, cada uno con un multiplicador sobre el tiempo de caminata y de guardado de equipaje:

  | Tipo | Proporción | Multiplicador |
  |---|---:|---:|
  | Discapacidad / embarazo (embarca antes que nadie) | 3% | ×2,0 |
  | Lento | 25% | ×1,5 |
  | Normal | 47% | ×1,0 |
  | Rápido | 25% | ×0,75 |

- **Tipo de vuelo:** en un vuelo **corto** `p(carry-on) = 0,75`, porque los pasajeros evitan despachar equipaje. En un vuelo **largo** `p(carry-on) = 0,35`.
- **Política nueva: *Premium-Prioridad*.** El orden es pre-embarque, después Primera Clase y después los Grupos 1 → 2 → 3. Dentro de cada grupo el orden es aleatorio o tipo Steffen.

### Resultados del bonus (N = 1.000)

| Política | Vuelo corto | Vuelo largo |
|---|---:|---:|
| Premium-Prioridad (aleatorio dentro del grupo) | 25,41 min | 21,96 min |
| Premium-Prioridad (Steffen dentro del grupo) | 23,71 min | 20,82 min |
| Random | 19,55 min | 17,25 min |
| **Steffen** | **16,26 min** | **15,20 min** |

**En tiempo, segmentar por clases es la opción más lenta**, porque genera más interferencias en el pasillo de las que evita. Ordenar a cada grupo con la lógica de Steffen reduce parte de esa pérdida.

### Trade-off: ¿entonces por qué las aerolíneas segmentan?

Asignamos tarifas por zona, tomando como referencia tarifas de cabotaje de Aerolíneas Argentinas: Primera €185, Grupo 1 €95, Grupo 2 €75 y Grupo 3 €63.

| Concepto | Valor |
|---|---:|
| Ingreso con tarifa única (100 × €75) | €7.500 |
| Ingreso con tarifas por zona | €9.796 |
| **Ingreso extra por segmentar** | **+€2.296** |
| Costo extra de demora (Premium-Prioridad con Steffen vs. Steffen, vuelo corto) | −€490 |
| **Beneficio neto de segmentar** | **≈ +€1.800 por vuelo** |

El ingreso extra supera en unas 4-5 veces el costo de embarcar más lento. Es la misma lógica de *revenue management* que usan las aerolíneas reales: **la segmentación por clases no se justifica por la velocidad de embarque, sino porque el ingreso de vender tarifas diferenciadas supera largamente su costo operativo.**

---

## Contenido del repositorio

| Archivo | Descripción |
|---|---|
| [`tp1_ACN.ipynb`](tp1_ACN.ipynb) | Notebook con todo el código, las explicaciones y los análisis: simulación, Monte Carlo, gráficos, modelo económico y bonus. |
| [`index.html`](index.html) | Presentación interactiva en formato consultoría. Tiene selector de escenarios, gráficos, tabla de resultados, calculadora de ahorro y la sección del bonus. Se abre directamente en el navegador. |
| [`Trabajo práctico 1 ACN 2026.pdf`](Trabajo%20práctico%201%20ACN%202026.pdf) | Enunciado del trabajo. |

### Estructura del código

```
Parámetros ─► crear_pasajeros() ─► ordenar_pasajeros(política) ─► simular()
                                                                    │
                        ┌───────────────────────────────────────────┤
                        ▼                                           ▼
          guardar_estado() + crear_html()                Monte Carlo (N corridas)
          animación HTML de una corrida            media, desvío, error estándar, IC 95%
                                                                    │
                                                                    ▼
                                                 gráficos + costo de demora + trade-offs
```

---

## Cómo correrlo

```bash
pip install numpy pandas matplotlib jupyter
jupyter notebook tp1_ACN.ipynb
```

Ejecutá las celdas en orden. Con `N_SIMULACIONES = 5000`, el Monte Carlo principal tarda alrededor de una hora en Python puro. Para una prueba rápida, bajá `N_SIMULACIONES` y `N_SIMULACIONES_BONUS`.

Al ejecutarse, el notebook genera **animaciones HTML autocontenidas** de una corrida por política (`boarding_<política>_animado.html`). En ellas se ve a cada pasajero caminando, guardando equipaje, esperando por interferencias y sentándose. Estos archivos pesan entre 10 y 20 MB cada uno, así que no están en el repositorio: se generan localmente. La presentación `index.html` los busca en una carpeta `simulaciones/`.

---

## Temas y competencias que muestra el trabajo

- **Simulación de eventos discretos:** diseño de una máquina de estados por agente, reglas de movimiento con restricciones de espacio (pasillo de una persona) y manejo de bloqueos entre agentes.
- **Métodos de Monte Carlo:** estimación de la media y del desvío estándar, error estándar, intervalos de confianza al 95% y análisis de convergencia 1/√N.
- **Análisis de sensibilidad:** cómo cambian los resultados según la proporción de pasajeros con carry-on (p = 0 / 0,5 / 1) y según el tipo de vuelo.
- **Modelado de heterogeneidad:** tipos de pasajeros con velocidades distintas y prioridades de embarque.
- **Traducción de resultados técnicos a decisiones de negocio:** costo de demora, costo de oportunidad de un asiento vacío y *revenue management* por segmentación de tarifas.
- **Visualización y comunicación:** gráficos de barras con IC, boxplots, gráfico de convergencia log-log, animaciones HTML propias y una presentación interactiva en formato consultoría.
- **Python científico:** `numpy`, `pandas`, `matplotlib` y generación de HTML/JS desde Python.
