ISCE2 example for processing a Sentinel-1C/D TOPS interferogram of the Southern Patagonian Icefield.

Download the SLC data and the precise orbits. Sentinel-1 data requires precise orbits (restituted [RESORB](https://s1qc.asf.alaska.edu/aux_resorb/) available same day or precise [POEORB](https://s1qc.asf.alaska.edu/aux_poeorb/), available three weeks later). You can get these files from 
```

wget https://datapool.asf.alaska.edu/SLC/SC/S1C_IW_SLC__1SDV_20260514T235559_20260514T235627_007656_00F8E9_4CEB.zip
wget https://datapool.asf.alaska.edu/SLC/SC/S1C_IW_SLC__1SDV_20260514T235625_20260514T235652_007656_00F8E9_450F.zip

wget https://s1qc.asf.alaska.edu/aux_poeorb/S1C_OPER_AUX_POEORB_OPOD_20260603T070923_V20260513T225942_20260515T005942.EOF
wget https://s1qc.asf.alaska.edu/aux_poeorb/S1D_OPER_AUX_POEORB_OPOD_20260604T071417_V20260514T225942_20260516T005942.EOF

```
Sentinel-1D data are available at the [Copernicus Browser](https://browser.dataspace.copernicus.eu) and do not have direct download links.

Create the input file `topsapp.xml` in the `20260514_20260515_s2` folder
```
<topsApp> 
	<component name="topsinsar"> 
		<property name="Sensor Name">SENTINEL1</property>
		<property name="demFilename">/Users/francisco/insarproc/dem/copernicus/cop_dem_glo30m_wgs84_icefields.dem</property>
        <property name="geocode demFilename">/Users/francisco/insarproc/dem/copernicus/cop_dem_glo30m_wgs84_icefields.dem</property>
 
	<component name="reference"> 
		<property name="safe">../S1C_IW_SLC__1SDV_20260514T235559_20260514T235627_007656_00F8E9_4CEB.zip, ../S1C_IW_SLC__1SDV_20260514T235625_20260514T235652_007656_00F8E9_450F.zip</property> 
		<property name="output directory">"reference"</property>              
 		<property name="orbit directory">/Users/francisco/insarproc/esa/s1orb</property> 
		<property name="auxiliary data directory">/Users/francisco/insarproc/esa/s1orb</property> 
	</component>
 
	<component name="secondary"> 
		<property name="safe">../S1D_IW_SLC__1SDV_20260515T235609_20260515T235637_002801_004BFD_CDB5.SAFE.zip, ../S1D_IW_SLC__1SDV_20260515T235634_20260515T235701_002801_004BFD_CD1C.SAFE.zip</property> 
		<property name="output directory">"secondary"</property>     
 		<property name="orbit directory">/Users/francisco/insarproc/esa/s1orb</property> 
		<property name="auxiliary data directory">/Users/francisco/insarproc/esa/s1orb</property> 
	</component>
 
	<property name="swaths">[2]</property>
	<property name="ESD coherence threshold">0.85</property>
	<property name="do ESD">True</property>
	<!-- <property name="extra ESD cycles">0</property> -->
	<property name="apply ionosphere correction">False</property> 
	<property name="do ionosphere correction">False</property> 
	<property name="consider burst properties in ionosphere computation">False</property> 
	<property name="azimuth looks">2</property>
	<property name="range looks">8</property>
	<property name="filter strength">0.5</property>
	<property name="do unwrap">True</property>
	<property name="unwrapper name">snaphu_mcf</property>
	<property name="geocode list">["merged/filt_topophase.unw","merged/filt_topophase.unw.conncomp","merged/phsig.cor"]</property>
<!-->	<property name="geocode list">merged/filt_topophase.flat</property>-->
	<property name="regionOfInterest">-47.54, -46.45, -73.96, -73.22</property>


 
 </component>
 </topsApp>


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


<img style="float: center;" src="filt_topophase_chn.flat.geo.png" style="width:100px;">







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




