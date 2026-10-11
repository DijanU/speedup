# Lab 12 — Investigación previa (Partes 1 y 2)

CC3069 Computación Paralela y Distribuida · Estuardo Castro (23890) y Luis Padilla (23663).

Este documento se escribió **antes** de correr los experimentos: tiene los datos del equipo, el marco teórico, el modelo derivado y las predicciones. Las mediciones están en `lab12.ipynb` y la comparación en `reporte_lab12.md`.

---

## 1. Datos del equipo

Obtenidos con `sysctl` (la salida queda en la primera celda del notebook):

| Dato | Valor |
|---|---|
| Equipo | MacBook Pro, Apple **M5 Pro** |
| Núcleos físicos (`hw.physicalcpu`) | 15 |
| Núcleos lógicos (`hw.logicalcpu`) | 15 |
| Clúster 0 (`hw.perflevel0`) | 5 núcleos **"Super"** (los más rápidos), L2 compartida de 16 MB |
| Clúster 1 (`hw.perflevel1`) | 10 núcleos **"Performance"**, L2 compartida de 8 MB |
| Núcleos de eficiencia | 0 (en el M5 Pro, Apple reemplaza el clúster de eficiencia por un segundo nivel de núcleos de rendimiento) |
| Hyper-Threading / SMT | **No**: lógicos = físicos (15 = 15) |
| L1d por núcleo | 64 KB (`hw.l1dcachesize`, reportado para el nivel por defecto) |
| Memoria | 24 GB unificada |
| Compilador | Apple clang + `libomp` de Homebrew, `-O2` |

Consecuencias para interpretar resultados:

- **p_max = 15**. No hay SMT, así que cada hilo tiene un núcleo físico propio hasta p = 15; por encima de eso habría sobresuscripción (no se prueba).
- Los núcleos **no son iguales**: 5 son más rápidos que los otros 10. Con p ≤ 5 el sistema operativo probablemente usa solo núcleos "Super"; desde p = 6 empiezan a entrar los "Performance", más lentos. Con `schedule(static)` cada hilo recibe la misma cantidad de iteraciones, así que el hilo que cae en un núcleo lento marca el tiempo de todos.
- macOS no deja fijar hilos a núcleos (no hay `OMP_PROC_BIND` efectivo ni afinidad), así que el planificador decide dónde corre cada hilo.
- Un solo hilo puede correr a mayor frecuencia que cuando todos los núcleos están ocupados (límite de potencia/temperatura del chip). Eso **favorece a Ts** y baja el speedup.

---

## 2. Marco teórico

Notación de clase: Ts = tiempo de la versión secuencial sin OpenMP; Tp = tiempo con p hilos.

| Medida | Fórmula | Qué responde |
|---|---|---|
| Speedup | S = Ts / Tp | ¿cuántas veces más rápido? |
| Eficiencia | E = S / p | ¿qué fracción de cada hilo se aprovechó? |
| Costo | C = p · Tp | ¿cuántos hilo-segundos pagué? |
| Efectividad | F = S / C = S / (p·Tp) = E / Tp | rapidez por unidad de costo |

Las cuatro fugas de eficiencia:

- **η_f** — parte secuencial: lo que no se puede repartir.
- **η_c** — comunicación y sincronización: crear/despertar hilos, barreras, combinar el `reduction`.
- **η_b** — desbalance: un hilo trabaja más que otros y los demás esperan.
- **η_r** — trabajo redundante: trabajo que la versión paralela hace y la secuencial no.

### 2.1 Primer modelo (pregunta 1.3): fracción paralelizable fija

Un programa tarda T1 en un núcleo. Una fracción fp de ese tiempo se reparte entre p hilos y el resto (1 − fp) no:

```
Tp = (1 − fp)·T1 + fp·T1 / p
```

Speedup:

```
S(p) = T1 / Tp = T1 / [ (1 − fp)·T1 + fp·T1/p ]
     = 1 / [ (1 − fp) + fp/p ]
```

T1 desaparece: el speedup depende solo de fp y p. Es la **ley de Amdahl**.

**Límite (pregunta 1.5).** Cuando p → ∞, fp/p → 0:

```
S(∞) = 1 / (1 − fp)
```

Con fp = 0.90, S(∞) = 1 / 0.10 = **10**. Con 1000 núcleos: S = 1 / (0.10 + 0.90/1000) = 1 / 0.1009 ≈ **9.91**. Pasar de 15 a 1000 núcleos solo sube el techo de 6.25 a 9.91.

**Despeje de fp (pregunta 1.6).** De 1/S = (1 − fp) + fp/p:

```
1/S = 1 − fp·(1 − 1/p)   ⇒   fp = (1 − 1/S) / (1 − 1/p)
```

Con S medido, este fp es la fracción paralelizable *efectiva*: si sale menor que el fp real del código, la diferencia es tiempo perdido que se comporta como si fuera secuencial (overhead, desbalance entre núcleos rápidos y lentos, frecuencia más baja con todos los núcleos activos).

**Efectividad con este modelo.** F = S / (p·Tp). Como Tp = T1·[(1 − fp) + fp/p]:

```
F(p) = 1 / ( T1 · p · [(1 − fp) + fp/p]² )
```

Derivando respecto de p y despejando, F es máxima cuando p·(1 − fp) = fp, es decir **p* = fp / (1 − fp)**. Con fp = 0.90, p* = 9; con fp = 0.75, p* = 3; con fp = 0.50, p* = 1; con fp = 0.95, p* = 19 (> 15, así que F crece hasta p_max). Con fp = 1.00 el modelo da F = 1/(T1) constante: cualquier p vale lo mismo, y en la práctica gana el p más chico porque el overhead crece con p.

### 2.2 Pregunta 1.8 (jefe y servidor de 64 núcleos), solo con el modelo

Programa de 20 s con 2 s secuenciales: fp = 18/20 = 0.90.

| p | Tp = 2 + 18/p (s) | S | E | C = p·Tp (hilo·s) | F = S/C |
|---|---|---|---|---|---|
| 1 | 20.00 | 1.00 | 1.00 | 20.0 | 0.0500 |
| 8 | 4.25 | 4.71 | 0.59 | 34.0 | 0.1384 |
| 9 | 4.00 | 5.00 | 0.56 | 36.0 | **0.1389** |
| 16 | 3.13 | 6.40 | 0.40 | 50.0 | 0.1280 |
| 64 | 2.28 | 8.77 | 0.14 | 146.0 | 0.0601 |
| ∞ | 2.00 | 10.00 | 0 | ∞ | 0 |

Respuesta: con 64 núcleos el programa pasa de 20 s a 2.28 s (S ≈ 8.8), pero 50 de los 64 núcleos están ociosos en promedio (E = 14 %) y la efectividad cae a menos de la mitad de la óptima. Con 9 núcleos se logra S = 5 con la mejor efectividad, y con 16 ya se tiene el 73 % del speedup de 64. El techo es 10× aunque compre infinitos núcleos: si quiere más velocidad, conviene atacar los 2 s de lectura (disco más rápido, leer en paralelo por bloques o solapar lectura con cómputo) antes que comprar núcleos.

### 2.3 Modelo de dos fases de clase (pregunta 2.4)

Sumar n números con p hilos: cada hilo suma n/p (n/p − 1 sumas) y luego se combinan los parciales en un árbol (log2 p sumas):

```
Tp = (n/p − 1) + log2 p        (en unidades de "una suma")
```

Con n = 16, p = 4: Tp = 3 + 2 = 5 sumas, Ts = 15, S = 3. El modelo cuenta **solo sumas**: supone que lanzar los hilos, repartir, sincronizar y combinar no cuesta nada más que las sumas del árbol. En una máquina real despertar un hilo y pasar una barrera cuesta del orden de microsegundos, mientras que una suma cuesta menos de un nanosegundo: ese supuesto es el que se rompe (fuga **η_c**).

### 2.4 Lo que hace OpenMP en cada `parallel for` además de sumar (pregunta 2.5)

1. El hilo maestro entra a la región paralela y despierta (o crea, la primera vez) el equipo de p − 1 hilos del pool.
2. Cada hilo calcula su rango de iteraciones según el `schedule`.
3. Cada hilo inicializa su copia privada de `s` (= 0 por el `reduction`).
4. Suma su bloque.
5. Se combinan las copias privadas en `s` (atómicos o un árbol, según la implementación).
6. Barrera implícita al final del `for`: todos esperan al más lento.
7. Los hilos vuelven al pool (espera activa un rato, luego duermen).

Además, cada hilo trae las líneas de caché de su bloque del arreglo y del acumulador compartido, y en un chip con dos clústeres el combinado puede cruzar entre las cachés L2 de cada clúster.

### 2.5 El Ts honesto (pregunta 2.2)

Si se usa como Ts el programa **paralelo con 1 hilo**, el speedup sale **inflado**: la versión OpenMP con un hilo carga con el overhead de la región paralela y suele ser igual o más lenta que la secuencial pura, así que dividir entre ella da un numerador más grande. Es tramposo porque compara el programa paralelo contra una versión empeorada de sí mismo, no contra lo mejor que se puede hacer sin paralelismo, que es lo que el usuario tendría si no paralelizara.

### 2.6 Ancho de banda de memoria (pregunta 2.7, idea previa)

Sumar un `double` es 1 operación por cada 8 bytes leídos. Con N = 16.7 M el arreglo pesa 128 MB: no cabe en ninguna caché y todo viene de la memoria principal. El recurso compartido por todos los núcleos es el **ancho de banda de memoria** (y el bus/fabric que conecta los clústeres con ella). Cuando unos pocos hilos ya la saturan, agregar más no ayuda. Esto no es ninguna de las cuatro fugas: no es parte secuencial, ni sincronización, ni desbalance, ni trabajo redundante; es contención por un recurso de hardware compartido.

---

## 3. Predicciones (escritas antes de correr)

### Parte 1 — Amdahl (W = 400 000, p = 1…15)

Amdahl ideal con p = 15: S = 1 / [(1 − fp) + fp/15].

| fp | S ideal (p=2) | S ideal (p=4) | S ideal (p=15) | **Mi predicción S(15)** | p con F máxima (modelo) |
|---|---|---|---|---|---|
| 1.00 | 2.00 | 4.00 | 15.00 | **≈ 11** | cualquiera (en la práctica p bajo) |
| 0.95 | 1.90 | 3.48 | 8.82 | **≈ 7.5** | 15 |
| 0.90 | 1.82 | 3.08 | 6.25 | **≈ 5.5** | 9 |
| 0.75 | 1.60 | 2.29 | 3.33 | **≈ 3.1** | 3 |
| 0.50 | 1.33 | 1.60 | 1.88 | **≈ 1.8** | 1 |

Razonamiento de la predicción con fp = 1.00: el trabajo son 400 000 unidades de ~1 µs ≈ 0.4 s, así que el overhead de crear hilos (microsegundos) es despreciable. Lo que más va a pesar es que 10 de los 15 núcleos son "Performance" y no "Super": con `schedule(static)` todos reciben el mismo número de iteraciones y el tiempo lo marca el núcleo más lento. Si un núcleo "Performance" rinde ~75 % de uno "Super", S(15) ≈ 15 × 0.75 ≈ 11. Espero la curva casi ideal hasta p = 5 y luego un quiebre.

### Parte 2a — Ts honesto

Predicción: t_seq y t_par(p=1) van a ser casi iguales (diferencia < 2 %), porque con 0.4 s de trabajo el costo de abrir una región paralela es invisible. Si hay diferencia, espero que la versión secuencial sea un poco más rápida.

### Parte 2b — Suma

Predicción: con N = 16 la versión paralela va a ser **mucho más lenta** (S ≈ 0.01–0.05, 20–100× más lenta): 16 sumas son ~10 ns y despertar 15 hilos más una barrera es del orden de microsegundos. Creo que la paralela empieza a ganar entre **N ≈ 10 000 y N ≈ 65 536** (necesita que cada hilo tenga del orden de microsegundos de trabajo). Con N = 16.7 M espero S de 4–6 con p = 15, limitado por el ancho de banda de memoria, no por el número de núcleos.
