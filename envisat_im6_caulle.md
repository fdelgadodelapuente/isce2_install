ISCE2 example for processing an ENVISAT IM6 interferogram of the 2011-2012 Cordon Caulle eruption. The key difference is that this is a noisy data set so the processing is low spatial resolution. It is also IM6 bea,  not the standard IM2.

Create the input file `sm_envisat.xml` in the `20060823_20081031` folder 
```
<stripmapApp>
	<component name="insar">

       <property name="Sensor Name">ENVISAT</property>
	<property name="demFilename">demLat_N44_N46_Lon_W112_W109.dem.wgs84</property>
<!--	<property name="demFilename">/Users/francisco/insarproc/dem/copernicus/svz_90m/cop_dem_glo90m_wgs84_svz.dem</property>-->
		<property name="reference doppler method">useDOPIQ</property>
		<property name="secondary doppler method">useDOPIQ</property>
       <property name="range looks">4</property>
       <property name="azimuth looks">20</property>

	<component name="reference">
       <property name="IMAGEFILE">../ASA_IM__0CNPDE20060823_050258_000000182050_00320_23420_2023.N1</property>
       <property name="INSTRUMENT_DIRECTORY">/Users/francisco/insarproc/esa/envisat/ins</property>
       <property name="ORBIT_DIRECTORY">/Users/francisco/insarproc/esa/envisat/dor_vor</property>
       <property name="OUTPUT">reference</property>
	</component>

	<component name="secondary">
       <property name="IMAGEFILE">../ASA_IM__0CNPDE20081001_050242_000000212072_00320_34442_6892.N1</property>
       <property name="INSTRUMENT_DIRECTORY">/Users/francisco/insarproc/esa/envisat/ins</property>
       <property name="ORBIT_DIRECTORY">/Users/francisco/insarproc/esa/envisat/dor_vor</property>
       <property name="OUTPUT">secondary</property>
	</component>

	<property name="filter strength">0.5</property>
	<property name="do unwrap">True</property>
	<property name="unwrapper name">icu</property>
	<property name="geocode list">["interferogram/filt_topophase.unw","interferogram/filt_topophase_msk.unw","interferogram/filt_topophase.unw.conncomp","geometry/los.rdr","interferogram/phsig.cor","interferogram/topophase.cor"]</property>

</component>
</stripmapApp>
```

Download the SRTM DEM in the `20060823_20081031` folder
```
dem.py -a stitch -b 44 46 -112 -109 -r -s 1 -c -u http://step.esa.int/auxdata/dem/SRTMGL1 -f
```


ENVISAT data requires precise orbits and the instrument files. You need either DOR (DORIS) or VOR (verified final) orbits. VOR orbits are more accurate. You can get these files from  http://topex.ucsd.edu/gmtsar/tar/ORBITS.tar

You can process ENVISAT SLC data 
`<property name="Sensor Name">ENVISAT_SLC</property>`
Note that the ENVISAT raw filename starts with `ASA_IM__0` (LEVEL0) while the filename of ENVISAT SLC data starts with `ASA_IMS_1` (LEVEL1).



Run it with
```
stripmapApp.py sm_alos.xml --steps
```

Export to Google Earth
```
cd interferogram

mdx.py filt_topophase.unw.geo -kml filt_topophase.unw.geo.kml

mdx filt_topophase.unw.geo -s 5725 -CW -unwr4 -rhdr 22900 -cmap cmy -wrap 6.283185307179586 -P; convert out.ppm -transparent cyan filt_topophase.unw.geo.png
```
You should get the following file


<img style="float: center;" src="filt_topophase.unw.geo.png" style="width:300px;">
