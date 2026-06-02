# Procesamiento InSAR Time Series con ISCE2 TOPS Stack y MintPy


Workflow para generar interferogramas Sentinel-1 con ISCE2 y series de tiempo InSAR mediante MintPy.

## Entrar al directorio principal

```bash
cd /home/lgodoy/insar_curso/salar_ts
```

---

# Instrucciones del TOPS Stack Processor

Contenido de `cmd.sh`:

```bash
#!/bin/bash

stackSentinel.py \
    -s ~/insar_curso/salar_ts/SLC/ \
    -o ~/esa/s1orb \
    -a ~/esa/s1orb \
    -w ~/insar_curso/salar_ts/${1} \
    -c 6 \
    -O 6 \
    -d /home/lgodoy/dem/atacama/cop_dem_glo30m_wgs84_salar.dem \
    -n '2' \
    -z 2 \
    -r 8 \
    -f 0.1 \
    -W interferogram \
    -b '-23.8 -23.4 -68.7 -68.06' \
    -u snaphu \
    -m 20230701
```

## Parámetros principales

| Parámetro | Descripción |
|------------|-------------|
| `-s` | Directorio con las imágenes Sentinel-1 SLC. |
| `-o` | Directorio con órbitas precisas (`POEORB`) o restituidas (`RESORB`). |
| `-a` | Directorio con archivos auxiliares. Normalmente igual que `-o`. |
| `-w` | Directorio de trabajo. `${1}` corresponde al nombre entregado al script. |
| `-c 6` | Número de interferogramas conectados hacia adelante para cada adquisición. |
| `-O 6` | Número de interferogramas overlap usados para estimar NESD. |
| `-d` | DEM utilizado para remover fase topográfica y realizar geocoding. |
| `-n '2'` | Swath a procesar. Por defecto: `1 2 3`. |
| `-z 2` | Looks en azimuth. |
| `-r 8` | Looks en rango. |
| `-f 0.1` | Intensidad del filtro interferométrico. |
| `-W interferogram` | Flujo de procesamiento (`slc`, `correlation`, `interferogram`, `offset`). |
| `-b` | Bounding box geográfico en formato SNWE. |
| `-u snaphu` | Método de unwrapping (`SNAPHU` o `ICU`). |
| `-m 20230701` | Imagen maestra de referencia. |

Este comando crea la estructura de carpetas para el procesamiento.

---

# Crear carpeta de trabajo

```bash
sh cmd.sh topsstack_salar
```

---

# Copiar script SLURM

```bash
cp tops_slurm.sh topsstack_salar/.
cat tops_slurm.sh
```

Contenido de `tops_slurm.sh`:

```bash
#!/bin/bash

#SBATCH -J tops_stack
#SBATCH -p general
#SBATCH -n 1
#SBATCH -c 4
#SBATCH --mem=64000
#SBATCH --mail-user=fdelgado@uchile.cl
#SBATCH --mail-type=ALL
#SBATCH -o tops_slurm_%j.out
#SBATCH -e tops_slurm_%j.err

sh run_files/run_01_unpack_topo_reference
sh run_files/run_02_unpack_secondary_slc
sh run_files/run_03_average_baseline
sh run_files/run_04_extract_burst_overlaps
sh run_files/run_05_overlap_geo2rdr
sh run_files/run_06_overlap_resample
sh run_files/run_07_pairs_misreg
sh run_files/run_08_timeseries_misreg
sh run_files/run_09_fullBurst_geo2rdr
sh run_files/run_10_fullBurst_resample
sh run_files/run_11_extract_stack_valid_region
sh run_files/run_12_merge_reference_secondary_slc
sh run_files/run_13_generate_burst_igram
sh run_files/run_14_merge_burst_igram
sh run_files/run_15_filter_coherence
sh run_files/run_16_unwrap
```

Este script ejecuta los 16 pasos del TOPS Stack Processor.

---

# Ejecutar procesamiento

```bash
cd topsstack_salar

sbatch tops_slurm.sh
```

El procesamiento suele tardar aproximadamente **2–3 horas**.

---

# Exportar interferogramas a PNG

```bash
cd ~/insar_curso/salar_ts/topsstack_salar/merged/interferograms

csh ~/insar_curso/scripts/isce2png_no_filter.csh

scp -P 4603 -r \
    lgodoy@leftraru.nlhpc.cl:~/insar_curso/salar_ts/topsstack_salar/merged/interferograms/pngs .
```

---

# Procesamiento de series de tiempo con MintPy

```bash
cd ~/insar_curso/salar_ts/topsstack_salar

mkdir ts

cp ../smallbaselineApp.cfg ts/.

cd ts

smallbaselineApp.py smallbaselineApp.cfg
```

---

# Parámetros relevantes de MintPy

Dependiendo de la calidad de los interferogramas, pueden ajustarse los siguientes parámetros:

| Línea | Parámetro | Ejemplo |
|---------|------------|---------|
| 073 | Looks adicionales en Y | `mintpy.multilook.ystep = 2` |
| 074 | Looks adicionales en X | `mintpy.multilook.xstep = 2` |
| 086 | Interferogramas a excluir | `mintpy.network.excludeIfgIndex = [1,2,3,4,5,8,9,10,11,14,15,16,19,20,23]` |
| 122 | Pixel de referencia | `mintpy.reference.yx = [970,620]` |
| 227 | Corrección troposférica | `mintpy.troposphericDelay.method = height_correlation` |
| 236 | Modelo atmosférico | `mintpy.troposphericDelay.weatherModel = ERA5` |
| 257 | Rampa a remover | `mintpy.deramp = linear` |

---

# Visualización de resultados

Serie de tiempo:

```bash
tsview.py geo/geo_timeseries_tropHgt_ramp.h5
```

Velocidad promedio:

```bash
view.py geo/geo_velocity.h5
```

Velocidad con rango personalizado:

```bash
view.py geo/geo_velocity.h5 velocity -v -1.5 1.5
```

---

# Exportar velocidad promedio a KMZ

```bash
cd geo

save_kmz.py geo_velocity.h5 -v -1 1
```

Copiar KMZ al computador local:

```bash
scp -P 4603 -r \
    lgodoy@leftraru.nlhpc.cl:~/insar_curso/salar_ts/topsstack_salar/ts/geo/*kmz .
```
