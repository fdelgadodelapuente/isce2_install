# Correr un  modelo de Mogi y Okada 


Descargar ```Mogi_Aniakchak.zip```, ```inversion2026.zip```,```okada_pishan.zip``` de U-Cursos y [load_isce](https://github.com/fdelgadodelapuente/isce_utils/blob/main/load_isce.m)

Crear carpeta llamada insar y meterlas todas adentro.

Mover ```load_isce.m``` en inversion/scripts, reescribiendo el que hay ahi.

## Mogi volcan Aniakchak 2023

Añadir carpetas al path de Matlab

```
addpath('inversion/InSamp/')
addpath('inversion/scripts/')
addpath('inversion/resamp_sill/')
addpath('inversion/mogi_invert/')
addpath('inversion/')
```
Correr ```mogi_example/220905_230807_1swath/synth_mogi``` 

Para el downsampling, correr ```resamptool_isce_roipac``` . Para otros datos editar ```mogi_example/220905_230807_1swath/resamp_in.m``` y volver a correr. Las variables clave son ``` filename``` , ``` losfilename``` , ``` demf``` , ``` lambda``` , ``` savestructname``` , ``` zone``` . El bloque de texto que dice 

```
coords.x1=-158.52083333333334;
coords.y2=58.200833333333335;
coords.nx=1945;
coords.ny=2401;
coords.dx=0.0008333333333333334;
```
lo puede reemplazar por coords=NaN, ya que es para una version muy antigua del codigo que leia la metadata de los xml mediante utilidades de Linux.

para invertir, correr ``` lsqnl_mogi2_chisq_Jac.m```. Si usa la version ams nueva de matlab, y falla, comentar las linas 58-60 que solo cambian las coordenadas de la fuente de  UTM a geograficas. Eso lo puede hacer con Google Earth. 


## Okada terremoto de Pishan 2016

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

Para modificar para otros datos debe cambiar los inputs de ```okada_pishan/asc/resamp_in.m```

```
```
