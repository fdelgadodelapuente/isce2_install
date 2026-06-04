**RADARSAT-2 data**

RADARSAT-2 SLCs are provided with only 5 state vectors, and they need to be interpolated to at least 9. Sometimes you can process your SLC with the original number of state vectors. RADARSAT-2 processing was fully operational in ISCE 2.2.0 (July 2018). Afterwards modules required to run the orbit extension module for innacurate state vectors were not included in newer versions of the software.

Open `isce2-2.6.2/install/isce/components/isceobj/Sensor/Radarsat2.py`, or `isce2-2.6.2/components/isceobj/Sensor/Radarsat2.py` and recompile. Then comment

```
planet = self.frame.instrument.platform.planet
        orbExt = OrbitExtender(planet=planet)
        orbExt.configure
        newOrb = orbExt.extendOrbit(tempOrbit)

        for sv in newOrb:
            self.frame.getOrbit.addStateVector(sv)
```

and replace with

```
for sv in tempOrbit:
            self.frame.getOrbit.addStateVector(sv)
```

This skips the orbit extension calculation and works for data in the Wide Ultra Fine beams.

Input XML file.

```
<property name="Sensor Name">RADARSAT2</property>
<property name="reference doppler method">useDEFAULT</property>
<property name="secondary doppler method">useDEFAULT</property>

<property name="xml">../RS2_OK117640_PK1032616_DK972095_U16W2_20200421_094740_HH_SLC/product.xml</property>
<property name="tiff">../RS2_OK117640_PK1032616_DK972095_U16W2_20200421_094740_HH_SLC/imagery_HH.tif</property>
```
