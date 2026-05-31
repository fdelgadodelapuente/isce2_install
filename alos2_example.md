ISCE2 example for processing a single swath od an ALOS-2 interferogram of Aniakchak crater, Aleutians.

Download the data
```
wget https://cumulus.asf.earthdatacloud.nasa.gov/L1.1/ALOS2/ALOS2447461150-220905-WBSR1.1__A.zip
wget https://cumulus.asf.earthdatacloud.nasa.gov/L1.1/ALOS2/ALOS2497141150-230807-WBDR1.1__A.zip
```
Create the input file **sm_alos2.xml** in the folder **20220905_20230807**. You only need to specify a single swath
```
<stripmapApp>
	<component name="insar">

	<property name="Sensor Name">ALOS2</property>
	<property name="reference doppler method">useDEFAULT</property>
	<property name="secondary doppler method">useDEFAULT</property>
	<property name="demFilename"></property>
	<property name="range looks">4</property>
	<property name="azimuth looks">8</property>

	<component name="reference">
		<property name="IMAGEFILE">../ALOS2447461150-220905/IMG-HH-ALOS2447461150-220905-WBSR1.1__A-F3</property>
		<property name="LEADERFILE">../ALOS2447461150-220905/LED-ALOS2447461150-220905-WBSR1.1__A</property>
		<property name="OUTPUT">reference</property>
	</component>

	<component name="secondary">
		<property name="IMAGEFILE">../ALOS2497141150-230807/IMG-HH-ALOS2497141150-230807-WBDR1.1__A-F3</property>
		<property name="LEADERFILE">../ALOS2497141150-230807/LED-ALOS2497141150-230807-WBDR1.1__A</property>
		<property name="OUTPUT">secondary</property>
	</component>

	<property name="filter strength">0.5</property>
	<property name="do unwrap">True</property>
	<property name="unwrapper name">icu</property>
	<property name="geocode list">["ionosphere/dispersive.bil.unwCor.filt","interferogram/filt_topophase.unw","interferogram/filt_topophase.unw.conncomp","geometry/los.rdr","interferogram/phsig.cor","interferogram/topophase.cor"]</property>
	<!--<property name="geocode bounding box">[S,N,W,E]</property>-->
	<property name="regionOfInterest">[56.43, 58.04, -158.49, -157.06]</property>

</component>
</stripmapApp>
```

Run it with
```
stripmapApp.py sm_alos.xml --steps
```
