ISCE2 example for processing a Sentinel-1C/D TOPS interferogram of the Southern Patagonian Icefield.

Download the SLC data and the precise orbits. Sentinel-1 data requires precise orbits. You can get these files from https://s1qc.asf.alaska.edu/aux_poeorb/
```
<!--
wget https://datapool.asf.alaska.edu/SLC/SA/S1A_IW_SLC__1SSV_20150414T234157_20150414T234224_005486_007013_5C44.zip
wget https://datapool.asf.alaska.edu/SLC/SA/S1A_IW_SLC__1SDV_20150426T234151_20150426T234227_005661_007430_AFBE.zip

wget https://s1qc.asf.alaska.edu/aux_poeorb/S1A_OPER_AUX_POEORB_OPOD_20150505T123046_V20150413T225944_20150415T005944.EOF
wget https://s1qc.asf.alaska.edu/aux_poeorb/S1A_OPER_AUX_POEORB_OPOD_20150517T123037_V20150425T225944_20150427T005944.EOF
-->
```
Create the input file **topsapp.xml** in the **20260514_20260515_s2** folder
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








