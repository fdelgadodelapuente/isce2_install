ISCE2 example for processing a Sentinel-1A ascending ad descenfing interferograms of the Mw 6.3 2015 Pishan earthquake (teleseismic moment tensors from [http://geoscope.ipgp.fr/index.php/en/catalog/earthquake-description?seis=us10002n4w](Geoscope), [https://earthquake.usgs.gov/earthquakes/eventpage/us10002n4w/executive](USGS)).

Download the SLC data and the precise orbits. Sentinel-1 data requires precise orbits. You can get these files from https://s1qc.asf.alaska.edu/aux_poeorb/
```

```

Create the input file `topsapp_asc.xml` in the `20150724_20150630` folder
```
<topsApp> 
	<component name="topsinsar"> 
		<property name="Sensor Name">SENTINEL1</property>
		<property name="geocode demFilename">3arcsec/demLat_N37_N38_Lon_E077_E080.dem.wgs84</property>
 
	<component name="master"> 
		<property name="safe">../S1A_IW_SLC__1SSV_20150630T124055_20150630T124122_006603_008CD2_7965.zip</property> 
		<property name="output directory">"master"</property>                 
		<property name="orbit directory">/Applications/insar_software/esa/s1orb</property> 
		<property name="auxiliary data directory">/Applications/insar_software/esa/s1orb</property> 
	</component>
 
	<component name="slave"> 
		<property name="safe">../S1A_IW_SLC__1SSV_20150724T124056_20150724T124123_006953_0096A6_544B.zip</property> 
		<property name="output directory">"slave"</property> 
		<property name="orbit directory">/Applications/insar_software/esa/s1orb</property> 
		<property name="auxiliary data directory">/Applications/insar_software/esa/s1orb</property> 
	</component>
 
	<property name="swaths">[1]</property>
	<property name="ESD coherence threshold">0.85</property>
	<property name="do ESD">False</property>
	<!-- <property name="extra ESD cycles">0</property> -->
	<!-- <property name="do ionosphere correction">False</property> -->
	<property name="azimuth looks">5</property>
	<property name="range looks">20</property>
	<property name="filter strength">0.5</property>
	<property name="do unwrap">True</property>
	<property name="unwrapper name">icu</property>
	<property name="geocode list">["merged/filt_topophase.unw", "merged/filt_topophase.unw.conncomp", "merged/los.rdr", "merged/filt_topophase.flat", "merged/topophase.cor", "merged/phsig.cor"]</property>
	<property name="region of interest">[37.34,37.68,77.903,78.2]</property>
 
 
 </component>
 </topsApp>



```
Create the input file `topsapp_dsc.xml` in the `20150718_20150624` folder


```
<topsApp> 
	<component name="topsinsar"> 
		<property name="Sensor Name">SENTINEL1</property>
		<property name="geocode demFilename">demLat_N37_N38_Lon_E077_E079.dem.3alks_3rlks.wgs84</property>
 
	<component name="master"> 
		<property name="safe">../S1A_IW_SLC__1SSV_20150624T004905_20150624T004932_006508_008A3D_3A0D.zip, ../S1A_IW_SLC__1SSV_20150624T004930_20150624T004957_006508_008A3D_BE38.zip</property> 
		<property name="output directory">"master"</property>                 
		<property name="orbit directory">/Applications/insar_software/esa/s1orb</property> 
		<property name="auxiliary data directory">/Applications/insar_software/esa/s1orb</property> 
	</component>
 
	<component name="slave"> 
		<property name="safe">../S1A_IW_SLC__1SSV_20150718T004905_20150718T004933_006858_0093F8_5FF1.zip, ../S1A_IW_SLC__1SSV_20150718T004931_20150718T004958_006858_0093F8_B2A4.zip</property> 
		<property name="output directory">"slave"</property> 
		<property name="orbit directory">/Applications/insar_software/esa/s1orb</property> 
		<property name="auxiliary data directory">/Applications/insar_software/esa/s1orb</property> 
	</component>
 
	<property name="swaths">[2]</property>
	<property name="ESD coherence threshold">0.85</property>
	<property name="do ESD">False</property>
	<!-- <property name="extra ESD cycles">0</property> -->
	<!-- <property name="do ionosphere correction">False</property> -->
	<property name="azimuth looks">5</property>
	<property name="range looks">20</property>
	<property name="filter strength">0.2</property>
	<property name="do unwrap">True</property>
	<property name="unwrapper name">icu</property>
	<property name="geocode list">["merged/filt_topophase.unw", "merged/filt_topophase.unw.conncomp", "merged/los.rdr", "merged/filt_topophase.flat", "merged/topophase.cor", "merged/phsig.cor"]</property>
	<property name="region of interest">[37.34,37.7,77.903,78.320]</property> <!--Caulle -->
 
 
 </component>
 </topsApp>

```


Run them with
```
topsApp.py topsapp.xml --steps
```

Export to Google Earth
```
cd merged

mdx.py filt_topophase.unw.geo -kml filt_topophase.unw.geo.kml

```

You should get the following file


<img style="float: center;" src="pishan_asc.flat.geo.png" style="width:100px;">




