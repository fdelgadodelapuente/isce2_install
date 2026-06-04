ISCE2 example for processing a PAZ interferogram of the Salar de Atacama basin.

We process the data with the Copernicus 30 m/pixel DEM to take advantaege of the high resolution of the X-band stripmap data (2 m/pixel in ground range). You can get the DEM from DLR and then you need to convert to the ISCE2 file format with GDAL.

Create the input file `sm_paz.xml` in the `20060823_20081031` folder 
```
<stripmapApp>
	<component name="insar">
	<property name="Sensor Name">COSMO_SKYMED_SLC</property>
   	<property name="demFilename">/home/fdelgado/dem/iceland/cop_dem_glo30m_wgs84_iceland_15m.dem</property> 
	<property name="reference doppler method">useDEFAULT</property>
	<property name="secondary doppler method">useDEFAULT</property>
	<property name="range looks">8</property> 
	<property name="azimuth looks">8</property> 

	<component name="reference">
		<property name="HDF5">../CSKS2_SCS_B_HI_12_HH_RD_SF_20240629193450_20240629193456.h5</property>
		<property name="OUTPUT">reference</property>
	</component>

	<component name="secondary">
		<property name="HDF5">../CSKS2_SCS_B_HI_12_HH_RD_FF_20240816193448_20240816193455.h5</property>
		<property name="OUTPUT">secondary</property>
	</component>

	<property name="filter strength">0.1</property>
	<property name="do unwrap">True</property>
	<property name="unwrapper name">icu</property>
	<property name="geocode list">interferogram/filt_topophase.unw, interferogram/phsig.cor, geometry/los.rdr</property>

</component>

</stripmapApp>

```

Run it with
```
stripmapApp.py sm_csk.xml --steps
```

Export to Google Earth
```
cd interferogram

mdx.py filt_topophase.unw.geo -kml filt_topophase.unw.geo.kml

mdx filt_topophase.unw.geo -s 3945-CW -unw -r4 -rhdr 15780 -cmap cmy -wrap 6.283185307179586 -P; convert out.ppm -transparent cyan filt_topophase.unw.geo.png
```
You should get the following file


<img style="float: center;" src="filt_topophase_paz.unw.geo.png" style="width:300px;">
