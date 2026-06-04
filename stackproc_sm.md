
## Stripmap Stack Processor

### ALOS raw data

Download the ALOS data from ASF to a folder called download, and unpack it with the ISCE parser.

```
prepRawALOS.py -i download/ -o SLC
prepRawSensor.py -i download/ -o SLC
```

If you do not want to apply the ionospheric correction, then zero pad the 14 MHz FBD images to match the 28 MHz FBS data. Otherwise, the parser will downsample the FBS data to FBD range sampling, so the split spectrum correction is applied to the common range bandwidth.

```
prepRawALOS.py -i download/ -o SLC --dual2single --fbd2fbs
```

Run the commands in `run_unPackALOS file`.

```
source ~/.bash_profile; unpackFrame_ALOS_raw.py -i  /home/fdelgado/insarproc/hudson/download/20070304  -o /home/fdelgado/insarproc/hudson/SLC/20070304 -f  fbs2fbd  -m

source ~/.bash_profile; unpackFrame_ALOS_raw.py -i  /home/fdelgado/insarproc/hudson/download/20071020  -o /home/fdelgado/insarproc/hudson/SLC/20071020 -m

source ~/.bash_profile; unpackFrame_ALOS_raw.py -i  /home/fdelgado/insarproc/hudson/download/20071205  -o /home/fdelgado/insarproc/hudson/SLC/20071205 -f  fbs2fbd  -m

source ~/.bash_profile; unpackFrame_ALOS_raw.py -i  /home/fdelgado/insarproc/hudson/download/20090309  -o /home/fdelgado/insarproc/hudson/SLC/20090309 -f  fbs2fbd  -m

source ~/.bash_profile; unpackFrame_ALOS_raw.py -i  /home/fdelgado/insarproc/hudson/download/20100125  -o /home/fdelgado/insarproc/hudson/SLC/20100125 -f  fbs2fbd  -m

source ~/.bash_profile; unpackFrame_ALOS_raw.py -i  /home/fdelgado/insarproc/hudson/download/20100612  -o /home/fdelgado/insarproc/hudson/SLC/20100612 -m

source ~/.bash_profile; unpackFrame_ALOS_raw.py -i  /home/fdelgado/insarproc/hudson/download/20100728  -o /home/fdelgado/insarproc/hudson/SLC/20100728 -m

source ~/.bash_profile; unpackFrame_ALOS_raw.py -i  /home/fdelgado/insarproc/hudson/download/20110128  -o /home/fdelgado/insarproc/hudson/SLC/20110128 -f  fbs2fbd  -m

source ~/.bash_profile; unpackFrame_ALOS_raw.py -i  /home/fdelgado/insarproc/hudson/download/20110315  -o /home/fdelgado/insarproc/hudson/SLC/20110315 -f  fbs2fbd  -m
```

To properly apply the split spectrum correction you need to process the data in the common spectral band, which means that all the FBS (28 MHz) images must be downsampled to the FBD (14 MHz) mode , so the pixel ratio is 4. If you process the data with `-f fbd2fbs` then the ionospheric correction might not work properly. In this example I process the data with 8 looks in range for the `FBD` pixel size, resulting in a pixel size of $\sim$ 160 m.

Run `stackStripmap.py`.

```
stackStripMap.py -W ionosphere -s SLC  -d /home/fdelgado/insarproc/dem/srtm_svz30m.dem  -m 20071205 -t 1500  -b 2000 -a 32 -r 8 -u snaphu -f 0.5
```

Here `-W` is the workflow, `-s` is the folder with the raw images extracted by the ISCE parser, `-d` is the DEM file name, `-m` is the reference image with respect to which all the other images will be aligned to, `-t` is the temporal baseline, `-a` is the azimuth looks, `-r` are the range looks, `-u` is the unwrapping module – either icu or snaphu, and `-f` is the power spectrum filtering strength.

`stackStripmap.py` will generate many configuration files that need to be manually executed. It also generates a perpendicular baseline plot `pairs.pdf` (`fig:bp`).

<figure id="fig:bp">
  <img src="figures/pairs.png" alt="pairs" width="900">
  <figcaption><strong>Figure.</strong> Perpendicular baseline plot for ALOS data. The reference image is not the earliest image.</figcaption>
</figure>


<figure id="fig:bp_alos114">
  <img src="figures/pairs_alos_p114.png" alt="pairs alos p114" width="900">
  <figcaption><strong>Figure.</strong> Perpendicular baseline plot for ALOS path 114. The reference image is not the earliest image. Note the systematic drift in the satellite orbit from 2007 to mid 2008 and then from late 2008 until the end of the mission in April 2011.</figcaption>
</figure>


The specific DEM files depend on the data set. Usually it is SRTM but if you have a better DEM like TanDEM-X 12 m you need to change it with the `-d` flag. Now run the following files

```
sh run_files/run_1_reference
sh run_files/run_2_focus_split
sh run_files/run_3_geo2rdr_coarseResamp
sh run_files/run_4_refinesecondaryTiming
sh run_files/run_5_invertMisreg
sh run_files/run_6_fineResamp
sh run_files/run_7_denseOffset
sh run_files/run_8_invertDenseOffsets
sh run_files/run_9_resampleOffset
sh run_files/run_10_replaceOffsets
sh run_files/run_11_fineResamp
sh run_files/run_12_grid_baseline
sh run_files/run_13_igram
sh run_files/run_14_igramLowBand
sh run_files/run_15_igramHighBand
sh run_files/run_16_iono
```

You can run them all with

```
ls  | awk '{print "sh",$1}' | csh
```

Run the stack processor, do not apply the dense offsets to improve the geometric coregistration, and apply the split spectrum correction.

```
sh run_files/run_1_reference
sh run_files/run_2_focus_split
sh run_files/run_3_geo2rdr_coarseResamp
sh run_files/run_4_refinesecondaryTiming
sh run_files/run_5_invertMisreg
sh run_files/run_6_fineResamp
sh run_files/run_7_denseOffset
sh run_files/run_8_invertDenseOffsets
sh run_files/run_9_resampleOffset
sh run_files/run_11_fineResamp
sh run_files/run_12_grid_baseline
sh run_files/run_13_igram
sh run_files/run_14_igramLowBand
sh run_files/run_15_igramHighBand
sh run_files/run_16_iono
```

Run the stack processor with split spectrum correction, and no dense offsets calculation.

```
sh run_files/run_1_reference
sh run_files/run_2_focus_split
sh run_files/run_3_geo2rdr_coarseResamp
sh run_files/run_4_refinesecondaryTiming
sh run_files/run_5_invertMisreg
sh run_files/run_6_fineResamp
sh run_files/run_12_grid_baseline
sh run_files/run_13_igram
sh run_files/run_14_igramLowBand
sh run_files/run_15_igramHighBand
sh run_files/run_16_iono
```

If you want to calculate the perpendicular baseline for all the processed interferograms, you can store them as text files in the `baselines` directoy

```
ls Igrams/202* -d1 | awk '{print "baseline.py -m SSC/SLCS/"substr($0,8,8),"-s SSC/SLCS/"substr($0,17,8)" | grep Baseline > baselines/ifg_"substr($0,8,8)"_"substr($0,17,8)}' > t.csh; csh t.csh
```

After several hours or days depending upon the computer, the software will output a set of unwrapped interferograms. The ionospheric correction is not applied automatically, so you can apply it with `imageMath.py`.

```
imageMath.py -e='a_0;a_1-b_0' -s BIL --a=Igrams/20071205_20100612/filt_20071205_20100612_snaphu.unw --b=Ionosphere/20071205_20100612/iono.bil.unwCor.filt  -o Igrams/20071205_20100612/filt_20071205_20100612_snaphu_nondispersive.unw 
```

You can also use awk to apply this command to all the interferograms

For ICU-unwrapped interferograms

```
ls Igrams/2*/*icu.unw | awk '{print "imageMath.py  -e=\47a_0;a_1*(c>0)-b_0*(c>0)\47 -s BIL  --a="$1,"--b=Ionosphere/"substr($0,8,18)"iono.bil.unwCor.filt","-o  "substr($0,1,51)"_nondispersive.unw --c="substr($0,1,47)".conncomp"}' 
```

For SNAPHU_MCF-unwrapped interferograms

```
ls Igrams/2*/*snaphu.unw | awk '{print "imageMath.py  -e=\47a_0;a_1*(c>0)-b_0*(c>0)\47 -s BIL  --a="$1,"--b=Ionosphere/"substr($0,8,18)"iono.bil.unwCor.filt", "-o  "substr($0,1,54)"_nondispersive.unw  --c="substr($0,1,47)"_snaphu.unw.conncomp"}'
```

<table>
<tr>
<td><img src="figures/dispersive.png" width="300"></td>
<td><img src="figures/ion.png" width="300"></td>
<td><img src="figures/nondispersive.png" width="300"></td>
</tr>
</table>

**Figure.** [a] ALOS interferogram of the Mw 7.0 Pichilemu earthquake with ionospheric streaks (`Igrams/20100309_20100424/filt_20100309_20100424_snaphu.unw`). [b] Ionospheric dispersive phase predicted by the split-spectrum correction that uses sub-band interferograms (`Ionosphere/20100309_20100424/iono.bil.unwCor.filt`). [c] Corrected interferogram after removal of the dispersive ionospheric phase (`Igrams/20100309_20100424/filt_20100309_20100424_snaphu_nondispersive.unw`).



Take looks on the line-of-sight file

```
looks.py -i merged/geom_reference/los.rdr -a 32 -r 8
```

The resulting file `los.32alks.8rlks.rdr` is the same one than `geometry/los.rdr` generated by `stripmapApp.py` when processing an interferogram with the same reference image and number of looks. You can also use the file `geom_reference/los.rdr` file. This file is in the ENVI file format instead of ISCE’s native RMG format, so you will have to convert it with `gdal_translate`.

```
gdal_translate los.rdr -Of ISCE los.r4
```

Geocode the relevant files

```
geocode.py -a 32 -r 8 -d /Users/francisco/insarproc/dem/srtm_svz90m.dem -m SLC/20071205 -i Igram/20161008_20161217/filt_20161008_20161217_snaphu.unw  -b -46.44 -45.63 -73.69 -72.48

geocode.py -a 32 -r 8 -d /Users/francisco/insarproc/dem/srtm_svz90m.dem -m SLC/20071205 -i Igram/20161008_20161217/20161008_20161217.cor  -b -46.44 -45.63 -73.69 -72.48

geocode.py -a 32 -r 8 -d /Users/francisco/insarproc/dem/srtm_svz90m.dem -m SLC/20071205 -i merged/geom_reference/los.32alks_8rlks.rdr -b -46.44 -45.63 -73.69 -72.48
```

Here `-b` is the geocoding bounding box and the rest of the parameters as in `stackStripmap.py`

You can also use `geocodeGdal.py`. Here the coordinates are for Kilauea, so you only have to change the bounding box for your area of interest. You can geocode to whatever posting you want. For example here I geocoded to a posting of 90 m/pixel or 3 arc seconds (1/1200 = 0.0008333333333333334).

```
geocodeGdal.py -l ../merged/geom_reference/lat.rdr -L ../merged/geom_reference/lon.rdr -f  20070620_20071221/filt_20070620_20071221_icu.unw  -b '19.2 19.6 -155.43 -154.93' -x 0.0008333333333333334 -y 0.0008333333333333334
```

A small set of the 6 ALOS images of the 2010 Eyjafjallajökull eruption without ionospheric corecction. Here I convert all the FBS to FBD images so they have a common range band for the split spectrum correction. I also use the open source [90 m TanDEM-X DEM](https://download.geoservice.dlr.de/TDM90/) because the default SRTM does not cover latitudes higher than 60 degrees, like those of Iceland. I stitched the DEM with GDAL and then I copied the relevant file dimensions to the .xml and .vrt files with the metadata.

```
unpackFrame_ALOS_raw_2.py  -i download/20071006 -o SLC/20071006
unpackFrame_ALOS_raw_2.py  -i download/20071121 -o SLC/20071121 -f fbs2fbd
unpackFrame_ALOS_raw_2.py  -i download/20080106 -o SLC/20080106 -f fbs2fbd
unpackFrame_ALOS_raw_2.py  -i download/20100413 -o SLC/20100413 -f fbs2fbd
unpackFrame_ALOS_raw_2.py  -i download/20100529 -o SLC/20100529 -f fbs2fbd
unpackFrame_ALOS_raw_2.py  -i download/20100714 -o SLC/20100714 -f fbs2fbd

stackStripMap.py -s SLC/ -d tdx90m.dem -t 1000 -b 1000 -a 16 -r 4 -u snaphu -W ionosphere -f 0.5 -m 20100413

geocode.py -a 16 -r 4 -d tdx90m.dem -m SLC/20100413 -i Igrams/20100413_20100714/filt_20100413_20100714_snaphu.unw -b 63.35 64.38 -20.16 -18.25
```

If you want to print the the metadata of a stripmap SLC with Python3

```
import shelve
s = shelve.open('data')
import isce
import isceobj
s2 = s['frame']
dir(s2)
```

### ERS, ENVISAT, COSMO-SkyMED raw data

The workflow is almost the same than for ALOS data. First, you need to dump the ENVISAT `ASA_IM_*.N1` and COSMO-SkyMED `CSKS*.h5` files into folders that have the image date as folder name. For TerraSAR-X you need to unzip each `TSX*SSC.tar.gz` file to extract the resulting `dims_op_oc_dfd2` folder. Then, unpack every image with

```
unpackFrame_ERS_raw.py  -i download/20070504 -o SLC/20070504 [-m]
unpackFrame_ENV_raw.py  -i download/20070504 -o SLC/20070504 [-m]
unpackFrame_CSK_raw.py  -i download/20070504 -o SLC/20070504
```

The `-m` flag is optional and allows you to merge different ENVISAT frames (if you need to stitch them and only for raw data). Now run `stackStripMap.py`. Here I use an example of ENVISAT IM6 data that have a pixel ratio of 3.

```
stackStripMap.py  -W interferogram  -s SLC -d /data/francisco/srtm_svz30m.dem -t 1500 -b 400 -a 20 -r 4  -u snaphu/icu -f 0.5 -m slcs/20050803
```

Then run the rest of the workflow as for ALOS raw data. As the X and C-band data are not really sensitive to the ionosphere, there is no need to run the split spectrum and rubber sheeting steps.

```
sh run_files/run_1_reference
sh run_files/run_2_focus_split
sh run_files/run_3_geo2rdr_coarseResamp
sh run_files/run_4_refinesecondaryTiming
sh run_files/run_5_invertMisreg
sh run_files/run_6_fineResamp
sh run_files/run_7_grid_baseline
sh run_files/run_8_igram
```




<figure id="fig:bp_envi_selected">
  <img src="figures/is2_a320.png" alt="is2 a320" width="900">
  <figcaption><strong>Figure.</strong> Perpendicular baseline plot for ENVISAT IM2 ascending track 320 at Yellowstone caldera ((Delgado2021a)) with selected interferograms and SLCs classified as either winter (blue circles) or non winter (red circles based on whether they result in low coherence interferograms or not. The figure does not include winter radar images. Note the satellite orbit spread from 2004 until early 2007. Afterwards the orbits were much better controlled.</figcaption>
</figure>


### ERS, ENVISAT, TerraSAR-X, COSMO-SkyMED, RADARSAT-2 and ALOS-2 stripmap SLC data

Unpack every image with `unpackFrame_SENSOR.py`. Here SENSOR is the name of the satellite. Unlike raw data, you cannot merge different stripmap frames if the data is provided as SLC because the different frames are focused with a specific set of parameters, including a Doppler centroid that is optimized for each image. For example, focusing parameters of a SLC might not be consistent with that of the following frame.

```
unpackFrame_ERS.py  -i download/20070504 -o SLC/20070504
unpackFrame_ENV.py  -i download/20070504 -o SLC/20070504
unpackFrame_CSK.py  -i download/20070504 -o SLC/20070504
unpackFrame_TSX.py  -i dims_op_oc_dfd2_661434447_1 -o SLC/20210221
unpackFrame_PAZ.py -i dims_op_oc_dfd2_661434447_1 -o SLC/20210221
unpackFrame_RSAT2.py -i tiff/20180119  -o SLC/20180119
unpackFrame_ALOS2.py -i download/20180119  -o SLC/20180119
unpackFrame_SAOCOM.py -i EOL1A/20220201_EOL1ASARSAO1B10547757 -o ~/saocom/colina/SLC/20220201
```

Run `stackStripmap.py`. The only difference is that here you will not focus the data, so you must set the software to read the data as zero-Doppler SLCs with the variable `-z`.

```
stackStripMap.py -W interferogram -z --nofocus -s slcs -m 20180315 -d /Users/francisco/Documents/dems/TanDEM-X/isce_dems/tandemx30m.dem -t 366 -b 200 -a 15 -r 15 -u snaphu 
```

The specific number of range and azimuth looks depends upon the satellite. The rest of the workflow is the same as in the previous examples.

<figure id="fig:bp_rs2">
  <img src="figures/pairs_rs2_u16w2.png" alt="pairs rs2 u16w2" width="900">
  <figcaption><strong>Figure.</strong> Perpendicular baseline plot for RADARSAT-2 data from beam U16W2 ((Delgado2021)).</figcaption>
</figure>


<figure id="fig:bp_csk">
  <img src="figures/pairs_csk_villarrica.png" alt="pairs csk villarrica" width="900">
  <figcaption><strong>Figure.</strong> Perpendicular baseline plot for COSMO-SkyMed descending data for Villarrica volcano ((Delgado2017)). Note the large spread in baselines compared with either ENVISAT (fig:bp_envi) or RADARSAT-2 (fig:bp_rs2). A comparison with an ALOS track is not direct because the ALOS missions had a systematic orbit drift and reset (fig:bp_alos114) that was not enforced for other satellites.</figcaption>
</figure>


<figure id="fig:bp_csk">
  <img src="figures/alos2_saocom_Bp.png" alt="alos2 saocom Bp" width="900">
  <figcaption><strong>Figure.</strong> Perpendicular baseline plot for ALOS-2 SM3 and SAOCOM-1 stripmap data ((Delgado2024)). SAOCOM-1 lacks a controlled orbital tube, so that results in large baselines compared with ALOS-2.</figcaption>
</figure>


