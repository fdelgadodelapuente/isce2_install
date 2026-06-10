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

abrir ```okada_pishan/asc/resamp_in.m```

Modificar linea 11 a ```lambda='sentinel'``` y comentar lineas 40-46
