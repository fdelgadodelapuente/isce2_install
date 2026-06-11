# Invertir modelos de Mogi y Okada 

El procedimiento de ajuste de un modelo a datos es un problema general de optimización mediante minimización de minimos cuadrados no lineales de la función $$\chi^2(\mathbf{m})$$ ($$\qquad (1)-\qquad (2) $$):

$$\min_{\mathbf{m}}  \chi^2(\mathbf{m}) = \sum_{i=1}^{N} \left( \frac{G_i(\mathbf{m}) - d_i} {\sigma_i} \right)^2 \qquad (1)$$

$$ \chi^{2}(\mathbf{m}) = \left[\mathbf{G}(\mathbf{m})-\mathbf{d}\right]^{T} \mathbf{C}_{d}^{-1} \left[\mathbf{G}(\mathbf{m})-\mathbf{d}\right] \qquad (2)$$

$$\mathbf{C}_d = \mathrm{diag} \left( \sigma_1^2, \sigma_2^2, \ldots, \sigma_N^2 \right) \qquad (3)$$

$$\mathbf{C}_d^{-1} = \mathrm{diag} \left( \frac{1}{\sigma_1^2}, \frac{1}{\sigma_2^2}, \ldots, \frac{1}{\sigma_N^2} \right) \qquad (4)$$

con $$\mathbf{m}$$ los parametros del modelo, $$\mathbf{G}$$ la funcion  (Okada/Mogi proyectado en el LOS de InSAR), $$d$$ los datos, $$\sigma_i$$ la desviacion standard de cada medición, $$\mathbf{C}_d^{-1} $$ la inverza de la matriz de covarianza, que en este caso la asumimos diagonal. Entonces el problema a minizar es la diferencia entre los datos y el modelo escalado por la incertidumbre. O sea, que la predicción del modelo explique los datos mas precisos, como muestra la siguiente figura. 

<img style="float: center;" src="figures/peulik_jers.png" style="width:300px;">

Interferograma JERS de banda L del complejo volcanico Ugashik-Mount Peulik (Aleutians, Alaska) e interferograma sintetico predicho por el mejor modelo de Mogi que ajusta los datos en base a la formula $$\qquad (1)$$. Un ciclo completo de colores es una fringe interferometrica y corresponde a 11.8 cm de deformacion superficial en el LOS de InSAR ([figura 6.193](https://link-springer-com.uchile.idm.oclc.org/chapter/10.1007/978-3-642-00348-6_6#Sec216) de [Lu y Dzurisin, 2014](https://link-springer-com.uchile.idm.oclc.org/book/10.1007/978-3-642-00348-6)).

Para ajustar los datos, utilizará el algoritmo de Levenberg-Marquardt para hacer estos modelos (secciones 9.2-9.3 del libro de inversion de [Aster et al., 2013](https://www.sciencedirect.com/book/monograph/9780123850485/parameter-estimation-and-inverse-problems)). 

Descargue  [load_isce](https://github.com/fdelgadodelapuente/isce_utils/blob/main/load_isce.m) y ```Mogi_Aniakchak.zip```, ```inversion2026.zip```,```okada_pishan.zip``` de U-Cursos a una carpeta que llamaremos  ```insar```. Mueva  ```load_isce.m``` a ```inversion/scripts```, reescribiendo el que hay ahí. La carpeta ```inversion/load_isce201708``` es una versión muy vieja de este codigo y la puede borrar. 

## Mogi Volcán Aniakchak 2023

El modelo de Mogi predice el desplazamiento en superficie $$\mathbf{u}(x,y)$$ en direcciones EW, NS, Z y en el LOS de InSAR $$u_{\mathrm{LOS}}(x,y)$$

$$
\mathbf{u}(x,y) = \left(u_E,u_N,u_U\right)^{\mathsf T} =
\frac{(1-\nu)\Delta V}{\pi} \frac{1}{R^3}
\left(x-x_0,y-y_0,d\right)^{\mathsf T},
\qquad
R^2=(x-x_0)^2+(y-y_0)^2+d^2.
$$

$$
u_{\mathrm{LOS}}(x,y)=
\frac{(1-\nu)\Delta V}{\pi R^3}
\left[l_E(x-x_0)+l_N(y-y_0)+l_Ud\right].
$$

$$\Delta V$$ es la variacion de volumen, $$x_0$$, $$y_0$$, $$z_0$$ las coordenadas de la esfera, $$R$$ la distancia desde el centro de la esfera al punto de observacion, $$\nu$$ el modulo de Poisson que equivale a 0.25 para rocas volcanicas promedio,$$l_e$$, $$l_n$$, $$l_z$$ los cosenos direccionales del vector LOS de InSAR.  

Para el problema de inversion de minimos cuadrados de un modelo de Mogi $$\mathbf{G(m)} = u_{\mathrm{LOS}}(x,y,\mathbf{m})$$, $$\mathbf{m}=[x_0, y_0, z_0, \Delta V]^T$$, $$\mathbf{d} = U_{LOS}(x,y)$$. Notar que los puntos de observacion en x,y no son las coordenadas de cada pixel, son conocidas y no son los parametros del modelo a optimizar. 



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
