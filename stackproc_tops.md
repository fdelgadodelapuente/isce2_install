

## TOPS Stack Processor

The workflow is detailed in .

### Example from Sierra Negra volcano

Download DEM

```
dem.py -a stitch -b -2 0 -92 -90 -r -s 1 -c -u  http://step.esa.int/auxdata/dem/SRTMGL1/
```

Run the stack processor `stackSentinel.py`.

```
stackSentinel.py -s slcs/ -o /home/fdelgado/isce/esa/s1orb -a /home/fdelgado/isce/esa/s1orb  -w stackproc -c 6 -O 6 -d demLat_S02_N00_Lon_W092_W090.dem.wgs84  -z 2 -r 8 -f 0.2 -u icu -W interferogram -b '-0.99 -0.64 -91.34 -90.88' -n '1'
```

The data are stored in the `slcs` folder and the products are processed in the `stackproc` folder. If you change the latter, then you need to change it for the next commands.

Run the configuration files.

```
sh  stackproc/run_files/run_1_unpack_topo_reference
sh  stackproc/run_files/run_2_unpack_secondary_slc
sh  stackproc/run_files/run_3_average_baseline
sh  stackproc/run_files/run_4_extract_burst_overlaps
sh  stackproc/run_files/run_5_overlap_geo2rdr_resample
sh  stackproc/run_files/run_6_pairs_misreg
sh  stackproc/run_files/run_7_timeseries_misreg
sh  stackproc/run_files/run_8_geo2rdr_resample
sh  stackproc/run_files/run_9_extract_stack_valid_region
sh  stackproc/run_files/run_10_merge_reference_secondary_slc
sh  stackproc/run_files/run_11_merge_burst_igram
sh  stackproc/run_files/run_12_filter_coherence
sh  stackproc/run_files/run_13_unwrap
```

The relevant files are

```
stackproc/merged/interferograms/20180303_20180408/fine.cor
stackproc/merged/interferograms/20180303_20180408/fine.int
stackproc/merged/interferograms/20180303_20180408/filt_fine.int
stackproc/merged/interferograms/20180303_20180408/filt_fine.cor
stackproc/merged/interferograms/20180303_20180408/filt_fine.unw
stackproc/merged/geom_reference/los.rdr 
```

Geocode the relevant files

```
geocodeIsce.py -f 'stackproc/merged/interferograms/20180303_20180408/filt_fine.unw stackproc/merged/interferograms/20180303_20180514/filt_fine.unw stackproc/merged/interferograms/20180303_20180619/filt_fine.unw stackproc/merged/interferograms/20180408_20180514/filt_fine.unw stackproc/merged/interferograms/20180408_20180619/filt_fine.unw stackproc/merged/interferograms/20180514_20180619/filt_fine.unw stackproc/merged/interferograms/20180303_20180408/filt_fine.cor stackproc/merged/interferograms/20180303_20180514/filt_fine.cor stackproc/merged/interferograms/20180303_20180619/filt_fine.cor stackproc/merged/interferograms/20180408_20180514/filt_fine.cor stackproc/merged/interferograms/20180408_20180619/filt_fine.cor stackproc/merged/interferograms/20180514_20180619/filt_fine.cor stackproc/merged/geom_reference/los.rdr'  -r 8 -a 2 -d demLat_S02_N00_Lon_W092_W090.dem.wgs84 -b '-0.99 -0.64 -91.34 -90.88' -m stackproc/reference -s stackproc/reference
```

Here the `-m` and `-s` flags are the same because the reference and secondary images all share the geometry of the reference SLC. You can also try

```
geocodeIsce.py -f 'stackproc/merged/interferograms/20180407_20180501/filt_fine.unw' -r 8 -a 2 -d demLat_S02_N00_Lon_W092_W090.dem.wgs84 -b '-0.99 -0.64 -91.34 -90.88' -m stackproc/coreg_secondarys/20180407 -s stackproc/coreg_secondarys/20180501
```

Whatever the case you should get the same result because after coregistration the lookup table is the same for all the SLCs.

You can also run the geocoding all at once with awk. Just change the DEM filename and the geocode bounding box.

```
ls stackproc/merged/interferograms/*/*unw stackproc/merged/interferograms/*/filt_fine.cor stackproc/merged/geom_reference/los.rdr | awk '{print "geocodeIsce.py -f ",$1,"-r 8 -a 2 -d demLat_S02_N00_Lon_W092_W090.dem.wgs84 -b \47-0.99 -0.64 -91.34 -90.88\47 -m stackproc/reference -s stackproc/reference"}'
```

### Example from Cordon Caulle with one-year long interferograms

For Cordon Caulle S1 track 83 descending, swath 2, 4 connections due to the 24 and 48 day repeat periods.

```
stackSentinel.py -s slcs/ -n '2' -o /home/fdelgado/isce/esa/s1orb -a /home/fdelgado/isce/esa/s1orb  -w stackproc -c 4 -O 4 -d /home/fdelgado/insarproc/dem/tandemx12m_sandes.dem   -z 5 -r 19 -f 0.5 -u snaphu -W interferogram -b '-40.83 -39.26 -72.59 -71.6'
```

After generating the aligned SLCs you can re rerun `stackSentinel.py` to generate the files for creating one-year long interfeorgrams if you need to link summer to summer interferograms because your area of interest is decorrelated half of the year.

```
stackSentinel.py -s slcs/ -n '2' -o /home/fdelgado/isce/esa/s1orb -a /home/fdelgado/isce/esa/s1orb -w stackproc2 -c 30 -O 30 -d /home/fdelgado/insarproc/dem/tandemx12m_sandes.dem   -z 5 -r 19 -f 0.5  -u snaphu -W interferogram -b '-40.83 -39.26 -72.59 -71.6'
```

Now link the required files

```
cd stackproc2
ln -sf ../stackproc/stack 
ln -sf ../stackproc/coreg_secondarys
```

and manually run the required interferograns in `run_files/run_11_merge_burst_igram` with `SentinelWrapper.py`

```
SentinelWrapper.py -c /home/fdelgado/insarproc/cordon_caulle/dsc/stackproc2/configs/config_igram_20200224_20210218
```

The rest (unwrapping, geocoding, etc) is the same than in the previous examples.

if you want to remove swaths to save HD space

```
ls *zip | awk '{print "zip -d",$1,substr($1,1,67)".SAFE/measurement/*iw1*tiff",substr($1,1,67)".SAFE/measurement/*vh*tiff",substr($1,1,67)".SAFE/measurement/*iw3*tiff"}'
```

<figure id="fig:bp_s1">
  <img src="figures/s1p83dsc_bp.png" alt="s1p83dsc bp" width="900">
  <figcaption><strong>Figure.</strong> Perpendicular baseline plot for Sentinel-1 descending track 83 for Cordon Caulle volcano ((Delgado2021a)). I classified the images based upon the satellite that acquired them (either 1A or 1B), as winter images when acquired in the austral winter and resulting in decorrelated interferograms in the volcano, or images from non-winter seasons that were acquired under rainy conditions and also resulting in low quality interferograms. Note that this is not the output of the Sentinel-1 Stack Processor, but a MATLAB figure that uses the perpendicular baseline information from the stack processor and a manual inspection of thousands of interferograms (incredibly tedious but necessary in areas with seasonal low coherence).</figcaption>
</figure>


### Example for the 2020 Nima M$_W$ 6.4 earthquake, path 121 descending 

```
stackSentinel.py -s slcs/ -o /home/fdelgado/isce/esa/s1orb  -a /home/fdelgado/isce/esa/s1orb  -w stackproc -c 4 -O 4 -d   demLat_N32_N35_Lon_E085_E088.dem.wgs84 -z 5 -r 19 -f 0.3 -u icu -W interferogram -b '32.7 33.7 86.3 87.3' -n '3'  
```

```
ls stackproc/merged/interferograms/*/*unw stackproc/merged/interferograms/*/*cor  | awk '{print "geocodeIsce.py -f  ",$1,"-r 19 -a 5 -d 3arcsec/demLat_N32_N35_Lon_E085_E088.dem.wgs84 -b \47 32.55 33.852 86.24 87.323 \47 -m stackproc/reference -s stackproc/reference"}' > t.csh; csh t.csh; rm t.csh
```

```
geocodeIsce.py -f stackproc/merged/geom_reference/los.rdr -r 19 -a 5 -d 3arcsec/demLat_N32_N35_Lon_E085_E088.dem.wgs84 -b '32.55 33.852 86.24 87.323' -m stackproc/reference -s stackproc/reference
```

### Ionospheric correction

You should run this if you are working in an area with a low geomagnetic latitude (i.e., Northern and Central Andes). This only corrects the long-wavelength dispersive phase, not the burst discontinuities due to an azimuthal variation in the TEC. The correction for the last effect is only implemented in `topsApp.py` as of version 2.6.3.

Provide the `ion_param.txt` file to `stackSentinel.py`. Run steps from 1 to 16 as usual, and then run the following ones:

```
sh  stackproc/run_files/run_17_subband_and_resamp
sh  stackproc/run_files/run_18_generateIgram_ion
sh  stackproc/run_files/run_19_mergeBurstsIon
sh  stackproc/run_files/run_20_unwrap_ion
sh  stackproc/run_files/run_21_look_ion
sh  stackproc/run_files/run_22_computeIon
sh  stackproc/run_files/run_23_filtIon
sh  stackproc/run_files/run_24_invertIon
```

Apply the correction

```
ls -d1 merged/interferograms/????????_???????? | awk '{print "imageMath.py -e='"'"'a*exp(-1.0*J*(b-c))'"'"' --a=merged/interferograms/"substr($0,23,17)"/fine.int --b=ion_dates/"substr($0,23,8)".ion --c=ion_dates/"substr($0,32,8)".ion -s BIP -t cfloat -o "$1"/fine_nondisp.int"}' | sh
```

Backup uncorrected interferograms

```
ls merged/interferograms/????????_????????/fine.int | awk '{print "mv "substr($0,1,39)"/fine.int ",substr($0,1,39)"/fine_orig.int"}' | sh
```

Copy corrected interferograms

```
ls merged/interferograms/????????_????????/fine_nondisp.int | awk '{print "mv "substr($0,1,39)"/fine_nondisp.int ",substr($0,1,39)"/fine.int"}' | sh
```

And now filter, unwrap and geocode.
