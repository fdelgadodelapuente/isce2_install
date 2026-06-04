ISCE2 example for processing an ENVISAT interferogram of the 2007 Father's Day dike intrusion at Kilauea volcano.

Create the input file `sm_envisat.xml` in the `20070514_20070618` folder 
```
<stripmapApp>
        <component name="insar">

       <property name="Sensor Name">ENVISAT</property>
       <property name="reference doppler method">useDOPIQ</property>
       <property name="secondary doppler method">useDOPIQ</property>
       <property name="range looks">2</property>
       <property name="azimuth looks">10</property>
       <property name="demFilename">/home/lgodoy/insar_curso/kilauea/dem/demLat_N18_N21_Lon_W157_W154.dem.wgs84</property>

        <component name="reference">
       <property name="IMAGEFILE">../ASA_IM__0PNPDK20070514_081955_000000182058_00093_27201_3022.N1</property>
       <property name="INSTRUMENT_DIRECTORY">/home/lgodoy/esa/envisat/ins</property>
       <property name="ORBIT_DIRECTORY">/home/lgodoy/esa/envisat/dor_vor</property>
       <property name="OUTPUT">reference</property>
        </component>

        <component name="secondary">
       <property name="IMAGEFILE">../ASA_IM__0PNPDK20070618_081957_000000182059_00093_27702_3230.N1</property>
       <property name="INSTRUMENT_DIRECTORY">/home/lgodoy/esa/envisat/ins</property>
       <property name="ORBIT_DIRECTORY">/home/lgodoy/esa/envisat/dor_vor</property>
       <property name="OUTPUT">secondary</property>
        </component>

        <property name="filter strength">0.3</property>
        <property name="do unwrap">True</property>
        <property name="unwrapper name">icu</property>
        <property name="geocode list">["interferogram/filt_topophase.unw","interferogram/filt_topophase.unw.conncomp","geometry/los.rdr","interferogram/phsig.cor","interferogram/topophase.cor"]</property>
        <!-- ><property name="regionOfInterest">[19.26,19.42,-155.31,-155.01]</property> -->

</component>
</stripmapApp>

```

Download the SRTM DEM in the `20070514_20070618` folder
```
dem.py -a stitch -b 18 21 -157 -154 -r -s 1 -c -u http://step.esa.int/auxdata/dem/SRTMGL1 -f
```


ENVISAT data requires precise orbits and the instrument files. You can get these files from  http://topex.ucsd.edu/gmtsar/tar/ORBITS.tar

Run it with
```
stripmapApp.py sm_envisat.xml --steps
```

Export to Google Earth
```
cd interferogram

mdx.py filt_topophase.unw.geo -kml filt_topophase.unw.geo.kml

```
You should get the following file

<img style="float: center;" src="filt_topophase_kilauea.unw.geo.png" style="width:300px;">
