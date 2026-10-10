---
title: "Lab 12 — Evidencia (Partes 1 y 2)"
subtitle: "CC3069 · Salidas de terminal capturadas al ejecutar lab12.ipynb"
---

Repositorio: <https://github.com/DijanU/speedup>. Las salidas se capturaron al ejecutar `lab12.ipynb` (cada bloque empieza con el comando `$ ...`, y stderr incluye el checksum).

# Datos del equipo (sysctl)

```
$ sysctl -n machdep.cpu.brand_string hw.physicalcpu hw.logicalcpu
Apple M5 Pro
15
15
$ sysctl hw.perflevel0.name hw.perflevel0.physicalcpu hw.perflevel0.l2cachesize hw.perflevel1.name hw.perflevel1.physicalcpu hw.perflevel1.l2cachesize hw.l1dcachesize hw.memsize
hw.perflevel0.name: Super
hw.perflevel0.physicalcpu: 5
hw.perflevel0.l2cachesize: 16777216
hw.perflevel1.name: Performance
hw.perflevel1.physicalcpu: 10
hw.perflevel1.l2cachesize: 8388608
hw.l1dcachesize: 65536
hw.memsize: 25769803776
$ pmset -g batt | head -1
Now drawing from 'Battery Power'
```

# Compilación (make) y uso

```
$ make clean && make
rm -f lab12 *.png *_con_S_E.csv
clang -O2 -Wall -Xpreprocessor -fopenmp -I/opt/homebrew/opt/libomp/include -o lab12 lab12_speedup.c -L/opt/homebrew/opt/libomp/lib -lomp
$ ./lab12 || true
Uso:
  ./lab12 amdahl     [W=400000] [fp=0.90] [pmax=15]
  ./lab12 suma       [pmax=15]
  ./lab12 desbalance [static|dynamic|guided] [W=40000] [pmax=15]
  ./lab12 gustafson  [W_por_hilo=100000] [W_seq=20000] [pmax=15]
```

# Modo amdahl (5 corridas)

```
$ ../lab12 amdahl 400000 1.00 > amdahl_fp100.csv
modo,N,p,param,t_seq,t_par,t_parte_seq
amdahl,400000,1,fp=1.00,0.302829,0.301577,0.000000
amdahl,400000,2,fp=1.00,0.302829,0.153388,0.000000
amdahl,400000,3,fp=1.00,0.302829,0.102832,0.000000
amdahl,400000,4,fp=1.00,0.302829,0.076062,0.000000
amdahl,400000,5,fp=1.00,0.302829,0.060291,0.000000
amdahl,400000,6,fp=1.00,0.302829,0.050003,0.000000
amdahl,400000,7,fp=1.00,0.302829,0.042834,0.000000
amdahl,400000,8,fp=1.00,0.302829,0.037382,0.000000
amdahl,400000,9,fp=1.00,0.302829,0.036180,0.000000
amdahl,400000,10,fp=1.00,0.302829,0.043521,0.000000
amdahl,400000,11,fp=1.00,0.302829,0.045274,0.000000
amdahl,400000,12,fp=1.00,0.302829,0.040561,0.000000
amdahl,400000,13,fp=1.00,0.302829,0.046261,0.000000
amdahl,400000,14,fp=1.00,0.302829,0.054823,0.000000
amdahl,400000,15,fp=1.00,0.302829,0.042222,0.000000
(checksum 6.399e+12)

$ ../lab12 amdahl 400000 0.95 > amdahl_fp95.csv
modo,N,p,param,t_seq,t_par,t_parte_seq
amdahl,400000,1,fp=0.95,0.342519,0.303682,0.016578
amdahl,400000,2,fp=0.95,0.342519,0.160478,0.016517
amdahl,400000,3,fp=0.95,0.342519,0.113323,0.017211
amdahl,400000,4,fp=0.95,0.342519,0.086513,0.016279
amdahl,400000,5,fp=0.95,0.342519,0.072402,0.015784
amdahl,400000,6,fp=0.95,0.342519,0.065704,0.016033
amdahl,400000,7,fp=0.95,0.342519,0.064474,0.015474
amdahl,400000,8,fp=0.95,0.342519,0.061674,0.016175
amdahl,400000,9,fp=0.95,0.342519,0.058965,0.016031
amdahl,400000,10,fp=0.95,0.342519,0.054601,0.015722
amdahl,400000,11,fp=0.95,0.342519,0.044879,0.016980
amdahl,400000,12,fp=0.95,0.342519,0.041973,0.017042
amdahl,400000,13,fp=0.95,0.342519,0.040370,0.016959
amdahl,400000,14,fp=0.95,0.342519,0.047433,0.016993
amdahl,400000,15,fp=0.95,0.342519,0.046150,0.016747
(checksum 5.775e+12)

$ ../lab12 amdahl 400000 0.90 > amdahl_fp90.csv
modo,N,p,param,t_seq,t_par,t_parte_seq
amdahl,400000,1,fp=0.90,0.311342,0.304253,0.034058
amdahl,400000,2,fp=0.90,0.311342,0.167324,0.032879
amdahl,400000,3,fp=0.90,0.311342,0.120005,0.033243
amdahl,400000,4,fp=0.90,0.311342,0.104589,0.033467
amdahl,400000,5,fp=0.90,0.311342,0.088233,0.033354
amdahl,400000,6,fp=0.90,0.311342,0.075843,0.031975
amdahl,400000,7,fp=0.90,0.311342,0.070110,0.032607
amdahl,400000,8,fp=0.90,0.311342,0.065209,0.032258
amdahl,400000,9,fp=0.90,0.311342,0.062034,0.031727
amdahl,400000,10,fp=0.90,0.311342,0.059573,0.032305
amdahl,400000,11,fp=0.90,0.311342,0.058263,0.031934
amdahl,400000,12,fp=0.90,0.311342,0.055974,0.031702
amdahl,400000,13,fp=0.90,0.311342,0.059454,0.033290
amdahl,400000,14,fp=0.90,0.311342,0.056320,0.033018
amdahl,400000,15,fp=0.90,0.311342,0.059443,0.031882
(checksum 5.183e+12)

$ ../lab12 amdahl 400000 0.75 > amdahl_fp75.csv
modo,N,p,param,t_seq,t_par,t_parte_seq
amdahl,400000,1,fp=0.75,0.294995,0.296309,0.081443
amdahl,400000,2,fp=0.75,0.294995,0.188248,0.079693
amdahl,400000,3,fp=0.75,0.294995,0.154681,0.080995
amdahl,400000,4,fp=0.75,0.294995,0.134131,0.079230
amdahl,400000,5,fp=0.75,0.294995,0.125805,0.080490
amdahl,400000,6,fp=0.75,0.294995,0.117202,0.080298
amdahl,400000,7,fp=0.75,0.294995,0.112885,0.080054
amdahl,400000,8,fp=0.75,0.294995,0.107877,0.079163
amdahl,400000,9,fp=0.75,0.294995,0.104570,0.079205
amdahl,400000,10,fp=0.75,0.294995,0.102188,0.079457
amdahl,400000,11,fp=0.75,0.294995,0.099585,0.079361
amdahl,400000,12,fp=0.75,0.294995,0.097878,0.079393
amdahl,400000,13,fp=0.75,0.294995,0.096545,0.079285
amdahl,400000,14,fp=0.75,0.294995,0.098845,0.079169
amdahl,400000,15,fp=0.75,0.294995,0.103786,0.079149
(checksum 3.600e+12)

$ ../lab12 amdahl 400000 0.50 > amdahl_fp50.csv
modo,N,p,param,t_seq,t_par,t_parte_seq
amdahl,400000,1,fp=0.50,0.301377,0.302968,0.158464
amdahl,400000,2,fp=0.50,0.301377,0.231870,0.159664
amdahl,400000,3,fp=0.50,0.301377,0.206214,0.158229
amdahl,400000,4,fp=0.50,0.301377,0.196311,0.159617
amdahl,400000,5,fp=0.50,0.301377,0.189244,0.159080
amdahl,400000,6,fp=0.50,0.301377,0.183960,0.159076
amdahl,400000,7,fp=0.50,0.301377,0.180806,0.159500
amdahl,400000,8,fp=0.50,0.301377,0.177932,0.159251
amdahl,400000,9,fp=0.50,0.301377,0.175801,0.159234
amdahl,400000,10,fp=0.50,0.301377,0.174736,0.158949
amdahl,400000,11,fp=0.50,0.301377,0.174449,0.159769
amdahl,400000,12,fp=0.50,0.301377,0.173853,0.159709
amdahl,400000,13,fp=0.50,0.301377,0.171493,0.158482
amdahl,400000,14,fp=0.50,0.301377,0.173737,0.158980
amdahl,400000,15,fp=0.50,0.301377,0.180839,0.163656
(checksum 1.600e+12)
```

# Modo suma

```
$ ../lab12 suma > suma.csv
modo,N,p,param,t_seq,t_par,t_parte_seq
suma,16,1,R=100000,0.000000001,0.000000054,0
suma,16,2,R=100000,0.000000001,0.000004571,0
suma,16,4,R=100000,0.000000001,0.000009608,0
suma,16,8,R=100000,0.000000001,0.000055291,0
suma,16,15,R=100000,0.000000001,0.000108128,0
suma,1024,1,R=48828,0.000000434,0.000000570,0
suma,1024,2,R=48828,0.000000434,0.000005522,0
suma,1024,4,R=48828,0.000000434,0.000010220,0
suma,1024,8,R=48828,0.000000434,0.000058068,0
suma,1024,15,R=48828,0.000000434,0.000108760,0
suma,65536,1,R=762,0.000034196,0.000032782,0
suma,65536,2,R=762,0.000034196,0.000024152,0
suma,65536,4,R=762,0.000034196,0.000023372,0
suma,65536,8,R=762,0.000034196,0.000062964,0
suma,65536,15,R=762,0.000034196,0.000110939,0
suma,1048576,1,R=47,0.000562683,0.000550508,0
suma,1048576,2,R=47,0.000562683,0.000289364,0
suma,1048576,4,R=47,0.000562683,0.000158701,0
suma,1048576,8,R=47,0.000562683,0.000199318,0
suma,1048576,15,R=47,0.000562683,0.000191085,0
suma,16777216,1,R=3,0.008780003,0.008713007,0
suma,16777216,2,R=3,0.008780003,0.004485687,0
suma,16777216,4,R=3,0.008780003,0.002256632,0
suma,16777216,8,R=3,0.008780003,0.001236359,0
suma,16777216,15,R=3,0.008780003,0.001285950,0
(checksum 2.143e+07)
```

# graficar.py

```
$ python3 ../graficar.py amdahl_*.csv suma.csv
[amdahl] 75 filas
  -> amdahl_speedup.png
  -> amdahl_eficiencia.png
[suma] 25 filas
  -> suma_speedup.png
```

# Repeticiones de fp = 1.00 (variación entre corridas)

```
rep 1: S(p=1..15) = 1.00 1.98 2.95 3.88 4.80 5.84 6.76 7.73 8.64 9.64 10.64 11.51 12.36 12.59 9.67
rep 2: S(p=1..15) = 0.98 1.96 2.88 3.79 4.76 5.84 6.76 7.65 8.40 9.22 10.27 11.12 12.29 11.42 8.78
rep 3: S(p=1..15) = 1.00 1.99 2.98 3.88 4.86 5.79 6.61 7.66 8.52 9.54 10.57 11.34 12.02 12.02 9.60
```

# Gráficas

![amdahl_speedup.png](resultados/amdahl_speedup.png){width=85%}

![amdahl_eficiencia.png](resultados/amdahl_eficiencia.png){width=85%}

![suma_speedup.png](resultados/suma_speedup.png){width=85%}

Las 4 gráficas restantes (`desbalance_*`, `gustafson_*`) son de las Partes 3 y 4 (integrante B).