ISCE2 example for processing an ENVISAT IM2 interferogram of Yellowstone caldera.

Download the data
```
wget https://imaging.unavco.org/data/sar/lts/export/ENV1/320/891/ASA_IM__0CNPDE20060823_050258_000000182050_00320_23420_2023.N1?token=ff995ded609ff9ba2882ca0d73968786bb35b3152c25b6b7d44d33ca55a15e2e&expires=1780252042

wget https://imaging.unavco.org/data/sar/lts/winsar/ENV2/320/873/ASA_IM__0CNPDE20081001_050242_000000212072_00320_34442_6892.N1?token=d6cae2212376f62cdaf42bad0635c44360747bc7b39bb362653e7eee06360b49&expires=1780252097
```
Create the input file **sm_alos.xml** in the folder **20100309_20100424**
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
	<property name="geocode list">["ionosphere/dispersive.bil.unwCor.filt","interferogram/filt_topophase.unw","interferogram/filt_topophase.unw.conncomp","geometry/los.rdr","interferogram/phsig.cor","interferogram/topophase.cor"]</property>
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

Run it with
```
stripmapApp.py sm_alos.xml --steps
```

