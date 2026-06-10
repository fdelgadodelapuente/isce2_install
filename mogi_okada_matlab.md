# Correr un  modelo de Mogi y Okada 

## Okada terremoto de Pishan 2016

Descargar ```Mogi_Aniakchak.zip```, ```inversion2026.zip```,```okada_pishan.zip``` de Ucursos y [load_isce](https://github.com/fdelgadodelapuente/isce_utils/blob/main/load_isce.m)

Crear carpeta llamada insar y meterlas todas adentro.

Mover ```load_isce.m``` en inversion/scripts, reescribiendo el que hay ahi.

añadir carpetas al path de Matlab

```
addpath('inversion/InSamp/')
addpath('inversion/scripts/')
addpath('inversion/resamp_sill/')
addpath('inversion/')
```

Abrir ```okada_pishan/asc/resamp_in.m``` modificar linea 11 a ```lambda='sentinel'``` y comentar lineas 40-46

Correr ```okada_pishan/asc/okada_forward_int.m``` y modificar los parametros de entrada del modelo. La linea 20 cambia las coordenadas UTM del area de interes (linea 24 de ```okada_pishan/asc/resamp_in.m```)

Para el downsampling descomentar las lineas 40-46 de ```okada_pishan/asc/resamp_in.m``` y correr ```resamptool_isce_roipac```

Para invertir, con los parametros por defecto correr ```okada_pishan/invert_eq.m```

```
```
