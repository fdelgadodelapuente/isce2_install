ISCE2 example for processing a Sentinel-1 TOPS interferogram of the 2015 Calbuco eruption.

Download the SLC data and the precise orbits. Sentinel-1 data requires precise orbits. You can get these files from https://s1qc.asf.alaska.edu/aux_poeorb/
```
wget https://datapool.asf.alaska.edu/SLC/SA/S1A_IW_SLC__1SSV_20150414T234157_20150414T234224_005486_007013_5C44.zip
wget https://datapool.asf.alaska.edu/SLC/SA/S1A_IW_SLC__1SDV_20150426T234151_20150426T234227_005661_007430_AFBE.zip

wget https://s1qc.asf.alaska.edu/aux_poeorb/S1A_OPER_AUX_POEORB_OPOD_20150505T123046_V20150413T225944_20150415T005944.EOF
wget https://s1qc.asf.alaska.edu/aux_poeorb/S1A_OPER_AUX_POEORB_OPOD_20150517T123037_V20150425T225944_20150427T005944.EOF

```
Create the input file `topsapp.xml` in the `20150414_20150426` folder
```
<topsApp>
	<component name="topsinsar">

	<property name="Sensor Name">SENTINEL1</property>
	<property name="demFilename">demLat_S42_S40_Lon_W74_W71.dem.wgs84</property>
	<property name="demFilename">geocode demLat_S42_S40_Lon_W74_W71.dem.wgs84</property>

	<component name="reference">
       	<property name="safe">../S1A_IW_SLC__1SDV_20150426T234151_20150426T234227_005661_007430_AFBE.zip</property>
       	<property name="output directory">reference</property>
		<property name="orbit directory">../orb</property> 
		<property name="auxiliary data directory">../orb</property> 
	</component>

	<component name="secondary">
       	<property name="safe">../S1A_IW_SLC__1SSV_20150414T234157_20150414T234224_005486_007013_5C44.zip</property>
       	<property name="output directory">secondary</property>
		<property name="orbit directory">../orb</property> 
		<property name="auxiliary data directory">../orb</property> 
	</component>

	<property name="swaths">[1,2]</property>
	<property name="ESD coherence threshold">0.85</property> <!-- above 0.7-->
	<property name="do ESD">True</property>
	<property name="extra ESD cycles">0</property> 
	<property name="do ionosphere correction">False</property>
	<property name="apply ionosphere correction">False</property>
	<property name="consider burst properties in ionosphere computation">False</property>
	<property name="azimuth looks">5</property>
	<property name="range looks">19</property>
	<property name="filter strength">0.5</property>
	<property name="do unwrap">True</property>
	<property name="unwrapper name">snaphu_mcf</property> <!--icu/snaphu_mcf-->
	<property name="geocode list">["merged/filt_topophase.unw","merged/filt_topophase.unw.conncomp","merged/los.rdr","merged/topophase.cor","merged/phsig.cor"]</property>
	<property name="region of interest">[-41.49,-41.32,-72.91,-72.46]</property>
	<!-- <property name="geocode bounding box">[-41.8,-40.84,-73.79,-72.00]</property>-->

</component>
</topsApp>

```

Download the SRTM DEM in the `20150414_20150426` folder
```
dem.py -a stitch -b -42 -40 -74 -71 -r -s 1 -c -u http://step.esa.int/auxdata/dem/SRTMGL1 -f
```


Run it with
```
topsApp.py topsapp.xml --steps
```

Export to Google Earth
```
cd merged

mdx.py filt_topophase.unw.geo -kml filt_topophase.unw.geo.kml

```


You should get the following file


<img style="float: center;" src="s1_calbuco.png" style="width:100px;">


You can clean-up intermediate files with
```
rm -v fine_interferogram/IW?/burst_??.int fine_interferogram/IW?/burst_??.cor fine_offsets/IW?/azimuth*off fine_offsets/IW?/range*off fine_coreg/IW?/burst_??.slc geom_reference/IW?/???_??.rdr ESD/overlap_IW?_??.int coarse_coreg/overlaps/IW?/*slc coarse_offsets/overlaps/IW?/*.off coarse_interferogram/overlaps/IW?/burst_bot_??_??.int coarse_interferogram/overlaps/IW?/burst_top_??_??.int */*full
```

### Ionosphere correction

You should use this correction for TOPS data in regiones of low geomagnetic latitudes . To correct the long wavelength dispersive phase and burst overlap jumps in `topsApp.py`, add the following to the input file.

```
<property name="do ionosphere correction">True</property>
<property name="apply ionosphere correction">True</property>
<property name="consider burst properties in ionosphere computation">True</property>
```



### Extract amplitude SLCs

Must be run with the following options in the int.xml file

```
<property name="use virtual files">False</property>
<property name="geocodelist">[merged/amplitudes.bil]</property>  
```

```
#gdal_translate -of ENVI merged/reference.slc.full.vrt merged/reference.slc.full
#gdal_translate -of ENVI merged/secondary.slc.full.vrt merged/secondary.slc.full
imageMath.py -e='abs(a);abs(b)' --a=reference.slc.full --b=secondary.slc.full -t FLOAT -s BIL -o amplitudes.bil.full
looks.py -i amplitudes.bil.full -a ${alks} -r ${rlks} -o amplitudes.bil
#gdal_translate -of ENVI merged/amplitudes.bil.vrt merged/amplitudes.bil
```

### Dense offsets

One particular advantage of the TOPS mode with respect to other stripmap modes is the small pixel size in the slant range direction. Therefore if your interferogram is decorrelated due to large strain, you can still retrieve deformation from range offsets instead of interferometry (e.g., ). Due to the small pixel size in range, you can also extract more accurate range than azimuth offsets, which are particularly useful for glaciology.

[Box sizes for ampcor](https://raw.githubusercontent.com/parosen/Geo-SInC/7f89ccfa906e36c12726a663ee3d34c621797214/UNAVCO2021/4.4_Offset_stack_for_velocity_dynamics/support_files/offset_parameters.png).

```
<!-- Ssearch Window Size Height should be < 2 * Window Size Height-->
<property name="do denseOffsets">True</property>
<property name="Ampcor window width">40</property>
<property name="Ampcor window height">8</property>
<property name="Ampcor search window height">10</property>
<property name="Ampcor search window width">10</property>
<property name="Ampcor skip width">32</property>
<property name="Ampcor skip height">8</property>
```

Example for Pine Island glacier (Antarctica) from the [2021 UNAVCO ISCE Workshop.](https://github.com/parosen/Geo-SInC/blob/main/UNAVCO2021/4.4_Offset_stack_for_velocity_dynamics/nb_topsApp_offsets.ipynb)

Example for Pine Island glacier (Antarctica) from the [2023 EarthScope ISCE Workshop.](https://github.com/parosen/Geo-SInC/blob/main/EarthScope2023/4.3_Offset_stack_for_velocity_dynamics_with_autoRIFT/nb_dense_offsets.ipynb)

```
<property name="do denseoffsets">True</property>
    <property name="Filter window size">3</property>
    <property name="Ampcor window width">256</property>
    <property name="Ampcor window height">64</property>
    <property name="Ampcor search window width">40</property>
    <property name="Ampcor search window height">10</property>
    <property name="Ampcor skip width">128</property>
    <property name="Ampcor skip height">32</property>
```

Example for southern Patagonia Icefield (Chile/Argentina)

```
<property name="do denseoffsets">True</property>
    <property name="Filter window size">3</property>
    <property name="Ampcor window width">128</property>
    <property name="Ampcor window height">32</property>
    <property name="Ampcor search window width">40</property>
    <property name="Ampcor search window height">10</property>
    <property name="Ampcor skip width">16</property>
    <property name="Ampcor skip height">4</property>
```

Then multiply the range and azimuth offsets by their pixel sizes of 2.3 and 14.1 m, respectively. The offset tracking uncertainty is 1/5 to 1/10 of the pixel size, so this results in theoretical uncertainties of 0.23-0.45 m for range offsets, and 1.41-2.82 m for azimuth offsets.

<figure>
  <img src="figures/ampcor_tops.png" alt="ampcor tops" width="900">
  <figcaption><strong>Figure.</strong> Range and azimuth offsets for 12 day Sentinel-1 pair.</figcaption>
</figure>





