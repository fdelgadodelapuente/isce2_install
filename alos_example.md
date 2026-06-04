ISCE2 example for processing an ALOS interferogram of the 2010 Mw 7.0 Pichilemu earthquake, Chile.

Download the data
```
wget https://datapool.asf.alaska.edu/L1.0/A3/ALPSRP219546480-L1.0.zip
wget https://datapool.asf.alaska.edu/L1.0/A3/ALPSRP219546490-L1.0.zip
wget https://datapool.asf.alaska.edu/L1.0/A3/ALPSRP226256480-L1.0.zip
wget https://datapool.asf.alaska.edu/L1.0/A3/ALPSRP226256490-L1.0.zip
```

Create the input file `sm_alos.xml` in the `20100309_20100424`  folder
```
<stripmapApp>
	<component name="insar">

	<property name="Sensor Name">ALOS</property>
	<property name="reference doppler method">useDOPIQ</property>
	<property name="secondary doppler method">useDOPIQ</property>
	<property name="demFilename">demLat_S36_S34_Lon_W073_W071.dem.wgs84</property>
	<property name="range looks">4</property>
	<property name="azimuth looks">8</property>

	<component name="reference">
		<property name="IMAGEFILE">../ALPSRP219546480-L1.0/IMG-HH-ALPSRP219546480-H1.0__A, ../ALPSRP219546490-L1.0/IMG-HH-ALPSRP219546490-H1.0__A</property>
		<property name="LEADERFILE">../ALPSRP219546480-L1.0/LED-ALPSRP219546480-H1.0__A, ../ALPSRP219546490-L1.0/LED-ALPSRP219546490-H1.0__A</property>
		<!--<property name="RESAMPLE_FLAG">dual2single</property>-->
		<property name="OUTPUT">reference</property>
	</component>

	<component name="secondary">
		<property name="IMAGEFILE">../ALPSRP226256480-L1.0/IMG-HH-ALPSRP226256480-H1.0__A, ../ALPSRP226256490-L1.0/IMG-HH-ALPSRP226256490-H1.0__A</property>
		<property name="LEADERFILE">../ALPSRP226256490-L1.0/LED-ALPSRP226256490-H1.0__A, ../ALPSRP226256490-L1.0/LED-ALPSRP226256490-H1.0__A</property>
		<!--<property name="RESAMPLE_FLAG">dual2single</property>-->
		<property name="OUTPUT">secondary</property>
	</component>

	<property name="filter strength">0.5</property>
	<property name="do unwrap">True</property>
	<property name="unwrapper name">icu</property>
	<property name="geocode list">["ionosphere/dispersive.bil.unwCor.filt","interferogram/filt_topophase.unw","interferogram/filt_topophase.conncomp","geometry/los.rdr","interferogram/phsig.cor","interferogram/topophase.cor"]</property>
	<!--<property name="geocode bounding box">[S,N,W,E]</property>-->
	<!--<property name="regionOfInterest">[S,N,W,E]</property>-->

	<!--for ionospheric correction only-->
    <property name="do split spectrum">True</property>
    <property name="do dispersive">True</property>
    <property name="do rubbersheetingRange">False</property>
    <property name="do rubbersheetingAzimuth">False</property>
	<!-- <property name="dispersive filter kernel x-size">800</property>
   	<property name="dispersive filter kernel y-size">800</property>
   	<property name="dispersive filter kernel sigma_x">100</property>
    <property name="dispersive filter kernel sigma_y">100</property>
    <property name="dispersive filter kernel rotation">0</property>
    <property name="dispersive filter number of iterations">5</property>
    <property name="dispersive filter mask type">connected_components</property>
   	<property name="dispersive filter coherence threshold">0.6</property> -->

</component>
</stripmapApp>
```

Download the SRTM DEM in the `20100309_20100424`  folder
```
dem.py -a stitch -b -36 -33 -73 -71 -r -s 1 -c -u http://step.esa.int/auxdata/dem/SRTMGL1 -f
```

Run it with
```
stripmapApp.py sm_alos.xml --steps
```

Apply the split spectrum ionospheric correction with 
```
imageMath.py -e='a_0;a_1*(c>0)-b_0*(c>0)' -s BIL  --a=interferogram/filt_pophase.unw.geo  --b=Ionosphere/dispersive.bil.unwCor.filt.geo  -o  interferogram/filt_topophase_nondispersive.unw.geo  --c=interferogram/filt_topophase.conncomp.geo
```
You can apply the split spectrum correction in radar coordinates. To do so, repeat the instruction but without any .geo extension, and then geocode the resulting *interferogram/filt_topophase_nondispersive.unw* file

Export to Google Earth
```
cd interferogram

mdx.py filt_topophase_nondispersive.unw.geo -kml filt_topophase_nondispersive.unw.geo.kml

mdx filt_topophase_nondispersive.unw.geo -s 4267 -ch2 -r4 -dr 17068 -cmap CMY -wrap 6.28 -P; convert out.ppm -transparent cyan filt_topophase_nondispersive.unw.geo.png
```
You should get the following file


<img style="float: center;" src="filt_topophase_nondispersive.unw.geo.png" style="width:300px;">


You can then load the interferogram to MATLAB, remove the remaining ramp after masking the coseismic signal with my utilities (https://github.com/fdelgadodelapuente/isce_utils) and export it to a GMT grid. Here I have incoporated the GCMT double couple moment tensor and a [compilation of active and potentially active Chilean faults](https://doi.pangaea.de/10.1594/PANGAEA.922241). 


<img style="float: center;" src="alos_20100309_20100424.png" style="width:300px;">
