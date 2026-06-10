Descargar ```Mogi_Aniakchak.zip```, ```inversion2026.zip```,```okada_pishan.zip``` de Ucursos y [load_isce](https://github.com/fdelgadodelapuente/isce_utils/blob/main/load_isce.m)



crear carpeta llamada insar y meterlas todas adentro
poner load_isce en inversion/scripts

añadir carpetas al path de Matlab

addpath('inversion/InSamp/')
addpath('inversion/scripts/')
addpath('inversion/resamp_sill/')
addpath('inversion/')


abrir okada_pishan/asc/resamp_in.m

definir lambda='sentinel'; en linea 11
comentar lineas 40-46
