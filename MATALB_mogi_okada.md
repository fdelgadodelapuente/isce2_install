# Correr un  modelo de Mogi y Okada 

Utilizará el algoritmo de Levenberg-Marquardt para hacer estos modelos. EL problema a resolver es 
$$\[\min_{\mathbf{m}} \; f(\mathbf{m}) = \sum_{i=1}^{N} \left( \frac{G_i(\mathbf{m}) - d_i} {\sigma_i} \right)^2 \]$$
con ```m``` los parametros del modelo, ```G``` la funcion  (Okada/Mogi proyectado en el LOS de InSAR), ```d``` los datos y ```$$$\sigma_i$``` la desviacion standard de cada medici´on. Entonces el problema a minizar es la diferencia entre los datos y el modelo escalado por la incertidumbre.

Descargar ```Mogi_Aniakchak.zip```, ```inversion2026.zip```,```okada_pishan.zip``` de U-Cursos y [load_isce](https://github.com/fdelgadodelapuente/isce_utils/blob/main/load_isce.m)

Crear carpeta llamada ```insar``` y meterlas todas adentro. La carpeta ```inversion/load_isce201708``` es una versión muy vieja y no hay que usarla, por lo que la puede borrar. 

Mover ```load_isce.m``` en inversion/scripts, reescribiendo el que hay ahí.

## Mogi Volcán Aniakchak 2023

Añadir carpetas al path de Matlab

```
addpath('inversion/InSamp/')
addpath('inversion/scripts/')
addpath('inversion/resamp_sill/')
addpath('inversion/mogi_invert/')
addpath('inversion/')
```
Correr ```mogi_example/220905_230807_1swath/synth_mogi``` 

Para el downsampling, correr ```resamptool_isce_roipac``` . Para downsamplear otros datos cambiar  ```mogi_example/220905_230807_1swath/resamp_in.m``` y volver a correr. Las variables clave son ``` filename```, ``` losfilename```, ``` demf```, ``` lambda```, ``` savestructname```, ``` zone```, al igual que las líneas 36,37,39,40 que cambian las dimensiones de la dislocación que fuerza el downsampling a zonas con deformación en el algoritmo de [Lohman y Simons, 2005](https://agupubs.onlinelibrary.wiley.com/doi/10.1029/2004GC000841) y Appendix A de [Lohman et al., 2010](https://agupubs.onlinelibrary.wiley.com/doi/full/10.1029/2010JB007710).

El bloque de texto que dice 

```
coords.x1=-158.52083333333334;
coords.y2=58.200833333333335;
coords.nx=1945;
coords.ny=2401;
coords.dx=0.0008333333333333334;
```
lo puede reemplazar por ```coords=NaN```, ya que es para una versión muy antigua del código que leía la metadata de los .vrt de ISCE2 mediante grep de Linux.

Para invertir, correr ``` lsqnl_mogi2_chisq_Jac.m```. Puede cambiar los datos con las líneas 11-12.  Si usa MATLAB R2026 y falla, comentar las linas 58-60 que convierten las coordenadas de la fuente de UTM a geográficas. Eso lo puede hacer con Google Earth. 

## Okada Terremoto de Pishan 2016

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

Para modificar para otros datos debe cambiar los inputs de ```okada_pishan/asc/resamp_in.m``` de la misma forma que para Mogi. El downsampling es idéntico. 
