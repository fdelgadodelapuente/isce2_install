ISCE2 example for processing an ENVISAT IM2 interferogram of Yellowstone caldera.

Download the data
```
wget https://imaging.unavco.org/data/sar/lts/export/ENV1/320/891/ASA_IM__0CNPDE20060823_050258_000000182050_00320_23420_2023.N1?token=ff995ded609ff9ba2882ca0d73968786bb35b3152c25b6b7d44d33ca55a15e2e&expires=1780252042

wget https://imaging.unavco.org/data/sar/lts/winsar/ENV2/320/873/ASA_IM__0CNPDE20081001_050242_000000212072_00320_34442_6892.N1?token=d6cae2212376f62cdaf42bad0635c44360747bc7b39bb362653e7eee06360b49&expires=1780252097
```
Create the input file **sm_alos.xml** in the folder **20060823_20081031**
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
<!--	<property name="regionOfInterest">[-40.64,-40.41,-72.37,-72.01]</property>-->

</component>
</stripmapApp>
```

ENVISAT data requires precise orbits and the instrument files. You can get these files from  http://topex.ucsd.edu/gmtsar/tar/ORBITS.tar

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
