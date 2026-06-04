# ISCE2 Software Manual for Earth Science

## Contents

- [Introduction and Acknowledgements](#introduction-and-acknowledgements)
- [ISCE2 processors and data access](#isce2-processors-and-data-access)
- [Range and azimuth sampling frequencies and pixel sizes](#range-and-azimuth-sampling-frequencies-and-pixel-sizes)
- [Satellites data catalogs and technical speciications](#satellites-data-catalogs-and-technical-speciications)
- [ISCE installation](#isce-installation)
  - [macOS installation](#macos-installation)
  - [restart terminal after this](#restart-terminal-after-this)
  - [Linux installation](#linux-installation)
  - [CUDA support for Linux](#cuda-support-for-linux)
  - [Earthscope ISCE2 short courses](#earthscope-isce2-short-courses)
  - [Other software](#other-software)
- [Interferometric processing workflows](#interferometric-processing-workflows)
  - [stripmapApp.py](#stripmapapppy)
  - [topsApp.py](#topsapppy)
  - [alos2App.py](#alos2apppy)
  - [Miscellaneous issues](#miscellaneous-issues)
  - [DEM generation from TanDEM-X CoSSC data](#dem-generation-from-tandem-x-cossc-data)
- [Ancillary information required for InSAR processing](#ancillary-information-required-for-insar-processing)
  - [DEMs](#dems)
  - [ENVISAT and Sentinel-1A/B processing](#envisat-and-sentinel-1ab-processing)
- [Stack processors](#stack-processors)
  - [Stripmap Stack Processor](#stripmap-stack-processor)
  - [TOPS Stack Processor](#tops-stack-processor)
  - [ALOS-2 Stack Processor](#alos-2-stack-processor)
- [Clean-up directories](#clean-up-directories)
- [Time Series Analysis](#time-series-analysis)
  - [MintPy](#mintpy)
- [Example interferograms of volcanoes and earthquakes from ENVISAT, ALOS, Sentinel-1, and ALOS-2](#example-interferograms-of-volcanoes-and-earthquakes-from-envisat-alos-sentinel-1-and-alos-2)
  - [Volcanoes](#volcanoes)
  - [Earthquakes](#earthquakes)


<div class="center">

<figure>
  <img src="figures/isce2gmt.jpg" alt="isce2gmt" width="900">
  <figcaption><strong>Figure.</strong> Examples of multi constellation InSAR and SAR data from the 2022 Mauna Loa dike intrusion and eruption. The volcano erupted more than 0.23 km$^3$ of basaltic lava during the 14 day-long eruption (<a href="https://www.jpl.nasa.gov/images/pia25526-airborne-nasa-radar-maps-mauna-loa-lava-changes-in-hawaii">JPL, 2022</a>). A-B) Sentinel-1A unwrapped and wrapped descending interfeorgram. C-D) COSMO-SkyMed ascending range and azimuth offsets. E-F) COSMO-SkyMed unwrapped and wrapped ascending interferogram ((Delgado2024)). Interferograms and range offsets are sensitive to displacement in the across track direction while azimuth offsets are sensitive to displacement in the along track direction. Observations from range offsets and interferometry resolve the same displacement. Range offsets are one order of magnitude less sensitive to deformation than interferometry, but unlike phase they are not prone to aliasing due to large strain. InSAR and azimuth offsets can be decomposed to resolve for the three dimensional displacement field. Sentinel-1A data are open access provided by the European Space Agency while COSMO-SkyMed data were provided by the Hawaiian Volcanoes Supersite .%hosted by the EarthScope Consortium <a href="https://web-services.unavco.org/brokered/ssara/gui ">Seamless SAR Archive (SSARA)</a>.</figcaption>
</figure>


</div>

# Introduction and Acknowledgements

[ISCE](https://github.com/isce-framework/isce2) (InSAR Scientific Computing Environment) is an InSAR processing software developed by NASA’s Jet Propulsion Laboratory (JPL) and it is currently the best free available software of its kind. ISCE contains three processors for different data sets: `stripmapApp.py` for stripmap data, `topsApp.py` for Sentinel-1 TOPS (Terrain Observations by Progressive Scans) data and `alos2App.py` for ALOS-2 data. It also includes stack processors for the different data sets. ISCE can process almost any SAR data set of interest for earthquake, volcano, glacier and hydro geodesy, except for TanDEM-X CoSSC.

This document provides a quick guide on how to use the ISCE software with several examples in volcanology and active tectonics. These examples are taken from the scientific literature or from my own research in these fields. If you have never used the software, this manual does not replace a formal software training like that provided by the JPL engineers during the UNAVCO ISCE workshops. In this document I have tried to compile common issues that new users run into when using the software for the first time. I wrote this document over a time span of several years when I was either a grad student at Cornell University, postdoc at Institut de Physique du Globe de Paris or faculty at Universidad de Chile. It is based on my own experience teaching ISCE to other colleagues, students and postdocs. It is inspired by Matthew Pritchard’s [Open-source software for geodetic imaging: ROI_PAC for InSAR and pixel tracking.](http://www.geo.cornell.edu/eas/PeoplePlaces/Faculty/matt/pub/winsar/InSAR_textbook_for_web_2014.pdf) I want to thank to several JPL scientists and engineers (Paul Rosen, Piyush Agram, Eric Fielding, Heresh Fattahi, Cunren Liang, and Paul Lundgren) for teaching me how to use ISCE, for answering many technical questions, and guiding me through the software internals. Many thanks to Tara Shreve, Whyjay Zheng, Patricia Macqueen, and Diego Lobos for providing comments that have improved this document.

# ISCE2 processors and data access

<div id="tab:insarapp">

| **Satellite** | **Year** | **stripmapApp** | **topsApp** | **alos2App** |
| --- | --- | --- | --- | --- |
| ERS-1/2 raw/SLC | 1991-2012 | Y | N | N |
| JERS | 1992-1998 | N | N | N |
| RADARSAT-1 | 1995-2013 | Y | N | N |
| ENVISAT raw/SLC | 2002-2012/04 | Y | N | N |
| ENVISAT ScanSAR | 2002-2010 | N | N | N |
| ALOS-1 raw/SLC | 2006-2011/04 | Y | N | N |
| ALOS-1 ScanSAR | 2006-2011/04 | N | N | N |
| TerraSAR-X | 2007- | Y | N | N |
| TanDEM-X CoSSC | 2011- | Y\* | N | N |
| COSMO-SkyMED raw/SLC | 2011- | Y | N | N |
| RADARSAT-2 | 2007- | Y | N | N |
| Sentinel-1 SM | 2014/10- | Y | N | N |
| Sentinel-1 TOPS | 2014/10- | N | Y | N |
| ALOS-2 SM | 2014/09- | Y | N | Y |
| ALOS-2 ScanSAR | 2015/02- | Y\*\* | N | Y |
| ALOS-2 ScanSAR to SM | 2015/02- | N | N | Y |
| ALOS-2 MAI | 2015/02- | N | N | Y |
| PAZ | 2018/02 - | Y\*\*\* | N | N |
| SAOCOM-1A/B SM | 2018/10- | Y++ | N | N |
| RADARSAT Constellation Mission | 2019/06- | N | N | N |
| CSK Second generation | 2021- | N | N | N |
| LuTan-1 | 2022- | Y | N | N |
| ALOS-4 | 2025- | Y++ | N | N |
| NISAR | 2025- | N | N | N |

Processing capabilities of each of the ISCE workflows. All data sets are stripmap (SM) except those labeled as either spotlight, ScanSAR or TOPS. TerraSAR-X, RADARSAT-2, ALOS-2, Sentinel-1, and SAOCOM-1 data are all distributed as zero-doppler SLC data only. \*Only with JPL unreleased bistatic processor. \*\*Yes but for a single swath only, and cannot do operations that require to extract the data as bursts (remove azimuth non-overlap spectra, range split-spectrum). \*\*\*as TanDEM-X data for `stripmapApp.py` and with a patch for the stripmap stack processor with [PAZ Parser](https://github.com/isce-framework/isce2/files/7597811/unpackFrame_PAZ.zip) ( [ISCE forum](https://github.com/isce-framework/isce2/discussions/401)). ++ SAOCOM-1 data can be ingested in the stripmap stack processor and ALOS-4 SM3 data can be processed with `stripmapApp.py` and the stripmap stack processor with [patches](https://github.com/isce-framework/isce2/pull/982) made by Francisco Delgado.

</div>

ISCE can process all the data sets described in Table 1, but it is does not necessarily incorporate the best algorithms targeted for processing a specific type of data set. You can think of ISCE as the Swiss Army knife of all the open-source InSAR processing software available.

# Range and azimuth sampling frequencies and pixel sizes

The fundamental frequencies of SAR imaging are the carrier frequency (9.6, 5.4, and 1.2 GHz for X-, C-, and L-band), the range bandwidth *B*, and the pulse repetion frequency *PRF*. The last two frequencies control the pixel size.

The slant range pixel size is

$$\Delta X_{sr} = \frac{c}{2B}$$ and the ground range pixel size is

$$\Delta X_{sr_g} = \frac{c}{2Bsin(\theta)}$$ with $c$ the speed of light, $B$ the range bandwidth, and $\theta$ the look angle. The last two numbers are platform dependent.

The azimuth pixel size is

$$\Delta X_{a} = \frac{|V_s|}{PRF}$$ with $V_{s}$ the (slow) velocity in the along track direction (azimuth) of $\sim$7.6 km/s and $PRF$ the pulse repetition frequency. The $PRF$ and slow velocity are also platform dependent. This can be scaled to account for the azimuth pixel size projected onto the Earth’s surface

$$\Delta X_{a_g} = \frac{|V_s|}{PRF} \frac{R_e}{h+R_e}$$

Here $R_e$ and h are the Earth’s radius and satellite elevation.

We use these formulas to calculate the pixel size of several data sets (`tab:slcres`-`tab:slcres_dem`).

These formulas do not apply for the TOPS mode. Here, $\Delta R_{a}$ is shrunk 4 times with respect to the stripmap mode , and $\Delta R_{g}$ is fixed to 2.32 m regardless of the swath, even though the range bandwidth is different for every swath. The Sentinel-1 TOPS pixel size in `tab:slcres_dem` are from the [ESA Sentinel-1 User Guide](https://sentinels.copernicus.eu/web/sentinel/user-guides/sentinel-1-sar/resolutions/level-1-single-look-complex), [ESA Sentinel-1 User Guide 2](https://sentinel.esa.int/web/sentinel/technical-guides/sentinel-1-sar/products-algorithms/level-1/single-look-complex/interferometric-wide-swath) and the [Sentinel-1 Product Definition, section 7.8](https://sentinels.copernicus.eu/documents/247904/1877131/Sentinel-1-Product-Definition.pdf/6049ee42-6dc7-4e76-9886-f7a72f5631f3?t=1461673251000). The exact pixel ratio changes depending upon the S1 swath because the slant range and the look angle increase from near to far range. If you want to process swath 1, it is better to use a pixel ratio of 3, while for swaths 2 and 3 it is better to use 4. If you want to process the whole SLC, then use 4. For Sentinel-1 it is better to use [odd looks](http://earthdef.caltech.edu/boards/4/topics/2063?r=2423#message-2423). For example, instead of using 20 looks in range and 5 in azimuth to retrieve a square pixel with a posting similar to that of the 1 arcsec SRTM it is better to set them to 19 looks in range and 5 looks in azimuth. If you want to process the S1 data with a resolution of 30 m/pixel, the ISCE default range and azimuth looks are 7 (28 m) and 3 (42 m) respectively. The pixel size is clearly not square but it conserves the energy at the center of the pixel.

<div id="tab:slcres">

| **Satellite** | **beam/mode** | **$\Delta t$ \[days\]** | **$\lambda$ \[cm\]** | **Look angle (º)** | **B$_{p}$ \[MHz\]** | **PRF \[MHz\]** |
| --- | --- | --- | --- | --- | --- | --- |
| ENVISAT | IM2 | 35 | 5.56 | 22 | 19.2 | 1652 |
| ENVISAT | IM6 | 30 | 5.56 | 41 | 19.2 | 1741 |
| ALOS-1 | FBS/FBD | 46 | 23.8 | 34 | 28-14 | 2100 -2130 |
| Sentinel-1 | TOPS IW | 6-12 | 5.55 | 33-38-43 | 57-48-43 | 1717-1451-1685 |
| COSMO-SkyMed | HIMAGE | 1-16 | 3.1 | 28 - 45 | 158 - 98 | 3260 - 3000 |
| ALOS-2 | SM3 | n/a | 24 | 40 | 28 | 2200 |

Sampling frequencies of different SAR satellites. Here B$_p$ and PRF are the range bandwidth and the pulse repetition frequency that control the range and azimuth resolution respectively. IM6 was the default beam used to record data during the [ENVISAT extension phase](https://earth.esa.int/eogateway/missions/envisat/description). All data sets are stripmap mode except Sentinel-1 which is TOPS (Terrain Observations by Progressive Scans). The PRF can change between ALOS-1 images and even in the middle of a frame, and hence requires a modification of the coregistration methods to handle pixels of different azimuth resolutions. The look angles increases from the near to the far range slant range of the SLC. This effect is very noticeable in the Sentinel-1 TOPS data due to its wide swath acquisitions. The three numbers for TOPS data refers to swaths 1 to 3.

</div>

<figure id="fig:az_res">
  <img src="figures/SARaz.png" alt="SARaz" width="900">
  <figcaption><strong>Figure.</strong> B and PRF for several SAR data sets. There is an proportionality relation between the PRF and B except for the Wide Swath modes like S1 TOPS and RADARSAT-2 F0W2.</figcaption>
</figure>


<figure id="fig:rang_res_Lband">
  <img src="figures/range_res_Lband.png" alt="range res Lband" width="900">
  <figcaption><strong>Figure.</strong> B several L-band SAR data sets, in addition to Sentinel-1 TOPS ((Delgado2024)).</figcaption>
</figure>


<div id="tab:slcres_dem">

| **Satellite** | **beam/mode** | **$\Delta R_r$ (m)** | **$\Delta R_a$ (m)** | **Pixel Ratio** | **Looks** |
| --- | --- | --- | --- | --- | --- |
| ENVISAT | IM2 | 20 | 4 | 5 | 2-4 |
| ENVISAT | IM6 | 12 | 4 | 3 | 2-4 |
| ALOS-1 | FBS | 10 | 5 | 2 | 4-8 |
| ALOS-1 | FBD | 20 | 5 | 4 | 4-8 |
| Sentinel-1$^*$ swath 2 | TOPS | 4 | 14.1 | 4 | 2-5 |
| COSMO-SkyMed | HIMAGE | 2 | 2 | 1 | 5-15 |
| TerraSAR-X/PAZ | SM | 2 | 2 | 1 | 5-15 |
| ALOS-2 | SM3 | 10 | 5 | 2 | 2-8 |
| SAOCOM-1 | S3, S4 | 10 | 5 | 2 | 2-8 |
| **DEM** |  | **Resolution (m)** |  |  |  |
| SRTM 3 arcsec |  | 90 | 90 |  |  |
| SRTM 1 arcsec |  | 30 | 30 |  |  |
| [TanDEM-X/COP 3 arcsec](https://download.geoservice.dlr.de/TDM90/) |  | 90 | 90 |  |  |
| [TanDEM-X/COP 1 arcsec](https://tandemx-science.dlr.de) |  | 30 | 30 |  |  |
| [TanDEM-X 0.4 arcsec](https://tandemx-science.dlr.de) |  | 12 | 12 |  |  |
| Pléiades |  | 2-10 | 2-10 |  |  |

SLC images resolution by platform compared with typical available DEMs. $\Delta R_r$ ground range pixel size. $\Delta R_a$ azimuth pixel size. $^*$Many papers from ESA show that Sentinel-1 SLC data have a pixel size of 20 m in azimuth and 4-5 in range, but those are approximates numbers only! Here COP is the Copernicus DEM.

</div>

# Satellites data catalogs and technical speciications

**ESA (ERA-1/2, ENVISAT, Sentinel-1)**: [ESA Online Dissemination](https://esar-ds.eo.esa.int/oads/access/), [SSARA](https://web-services.unavco.org/brokered/ssara/gui), [ASF Vertex](http://vertex.daac.asf.alaska.edu), [PEPS](http://peps.cnes.fr)

**JAXA (ALOS, ALOS-2)**: [ASF Vertex](http://vertex.daac.asf.alaska.edu) (ALOS), [G-Portal](https://gportal.jaxa.jp) (ALOS-2 ScanSAR)

**DLR (TerraSAR-X / TanDEM-X)**: [DLR Supersites](https://download.geoservice.dlr.de/supersites/files/), [EOWEB](https://eoweb.dlr.de/egp/) (catalog)

**COSMO-SkyMed**: CEOS Volcano Demonstrator, CEOS Supersites (some are stored at [SSARA](https://web-services.unavco.org/brokered/ssara/gui)), [data catalog](https://portal.cosmo-skymed.it/)

**SAOCOM-1**: [CONAE data catalog](https://catalog.saocom.conae.gov.ar/catalog/) (for Argentina and parts of Chile).

ALOS-2 SM3 data require a EORA2/EORA3 proposal with JAXA. SAOCOM data outside of Argentina require a proposal with CONAE. RADARSAT-2 data is commercial.

[ESA Online Dissemination](https://esar-ds.eo.esa.int/oads/access/): ERS-1/2 raw and SLC, ENVISAT raw and SLC. Raw data is for PI only.

[UNAVCO SSARA](http://web-services.unavco.org/brokered/ssara/gui): ALOS, ENVISAT raw, ALOS-2 (PI only), mostly North America. It also has links to the ESA Supersites data base.

[Alaska Satellite Facility](https://vertex.daac.asf.alaska.edu): ALOS, Sentinel-1.

[ESA Copernicus SciHub](https://scihub.copernicus.eu/dhus/#/home): Sentinel-1

[CNES Plateforme d’Exploitation des Produits Sentinel (PEPS)](https://peps.cnes.fr): Sentinel-1

[DLR](https://eoweb.dlr.de/egp/): TerraSAR-X, TanDEM-X (PI only)

[DLR](https://tandemx-science.dlr.de): TanDEM-X DEM (PI only)

[DLR Supersites](https://download.geoservice.dlr.de/supersites/files/LatinAmerica/): data made available by CEOS Supersites, CEOS Volcano Demonstrator and so on. It includes some data I’ve ordered for Laguna del Maule, Chaiten, Lazufre, Cordon caulle, Sabancaya, and others. It also includes data from cool places like Hawaii and Iceland. The website is a mess because it only list the image file name – you need to know the SLC track, frame and date in advance from the EOWEB catalog.

[CSA EODMS](https://www.eodms-sgdot.nrcan-rncan.gc.ca/index-en.html): RADARSAT, RADARSAT-2

[JAXA G-Portal](https://gportal.jaxa.jp/gpr/): ALOS, ALOS-2

[JAXA satpf](https://satpf.jp/spf/): ALOS, ALOS-2

[JAXA AUIG4](http://auig4.jaxa.jp/app/en/home/): ALOS-4

[ASI](https://www.asi.it/en/earth-science/cosmo-skymed/): [COSMO-SkyMED, COSMO-SkyMED first and second generation specifications.](https://www.asi.it/wp-content/uploads/2021/03/CSG-Mission-and-Products-Description-defpdf-1.pdf)

[CONAE](https://catalog.saocom.conae.gov.ar/catalog/#/): SAOCOM-1

# ISCE installation

Unfortunately installing ISCE is not straightforward. This section assumes that you have some familiarity with the GNU compiler and installing Python libraries with tools like macports and conda. If you have never used these tools, it is very likely that the ISCE installation will fail over and over and you will not understand the errors.

## macOS installation

You can build the software with MacPorts, but I have not tested it.

```
sudo port install py37-isce2 +gcc7
```

Instead, I compile it manually. For installing ISCE on macOS I suggest strongly suggest you to stick to a single package manager (either macports, brew or conda). Otherwise you might find conflicting issues due to the different versions of the installed libraries. For macports you can follow these instructions that work for 2014 to 2026 computers. These instructions are adapted from <https://github.com/piyushrpt/mojaveSetup?tab=readme-ov-file>

```
### Francisco Delgado, IPGP, ISCE installation
### Feb 22 2019 MacMini2014/MacBookAir2015, High Sierra and Mojave, gcc7
### Nov XY 2021 MacMini2014 Monterey, python37 and gcc11 
### Feb 06 2025 MacBookAir2015 Monterey, python312 and gcc13
### Feb 07 2025 MacBookAir2015 Monterey, python39 and gcc11 after python312 failure in running stripmapApp. Default compiler in the mp system is gcc13, though
### Aug 07 2025 MacMini M4 Pro Sequoia, python313 and gcc13
### May 28 2026 MacBook Neo Tahoe, python313 and gcc13


#### first reinstall macports for the new OSX version, then
port -qv installed > myports.txt ### backup existing ports
port echo requested | cut -d ' ' -f 1 > requested.txt ### backup another thing
sudo port -f uninstall installed ### remove old ports
sudo rm -rf /opt/local/var/macports/build/*  ### remove old ports
curl --location --remote-name 
https://github.com/macports/macports-contrib/raw/reference/restore_ports/restore_ports.tcl
chmod +x restore_ports.tcl

#if starting from a new MacOS, install macports and then self update
sudo port selfupdate

### install Xcode, then install the Xcode tools with
xcode-select --install

### ISCE is written in FORTRAN, C, C++. Use the GNU compiler gcc 
sudo port install gcc13
sudo port select gcc mp-gcc13    
port select --list gcc ## check the versions of gcc installed
## restart terminal after this

### Install python3 and many libraries. Never use the default Python included in macOS
sudo port install python313
sudo port select python3 python313
sudo ln -s /opt/local/Library/Frameworks/Python.framework/Versions/3.13/include/python3.13m /opt/local/include/python3.13m  #not neede for python3.13 in MacMini Neo

sudo port install xorg-libXt +flat_namespace   #for mdx
sudo port install freetype tiff openmotif      #for mdx
sudo port install fftw-3 +gcc13   
sudo port install fftw-3-single +gcc13    
sudo port install hdf5 +gcc13    #FAILED, then skipped to install h5py Feb 2025
sudo port install hdfeos5 h5utils #FAILED, then skipped to install h5py Feb 2025
sudo port install py313-numpy +gcc13
sudo port install py313-scipy +gcc13 #FAILED, installed with +gcc flag
sudo port install py313-matplotlib +cairo
sudo port install py313-pandas    ###not sure if I really needed
sudo port install py313-cython
ln  -s /opt/local/bin/cython-3.13 /opt/local/bin/cython3
cython3 -V #### check cython3 version  > 0.28
sudo port install py313-h5py  #python binding for h5py
sudo port install py313-matplotlib-basemap #flag not existent anymore with 3.12 version
####sudo port install opencv +python36 #for ionospheric correction only
#### for Monterey update to opencv3, still called cv2 in python, though
#sudo port install opencv3 +python313
#sudo port install py313-opencv3   
#### for Monterey and python3.12 update to opencv4, still loaded as import cv2, though.
sudo port install opencv4 +python313
sudo port install py313-opencv4     #python binding for opencv4, it doesnt exist for opencv3 in the MacBookNeo

#### ---
sudo port install py313-ipython 
sudo port select --set ipython3 py313-ipython
sudo port install py313-jupyter 
sudo port install hdf4 hdfeos
######sudo port install postgresql95 postgresql95-server    #this wasn't installed
#sudo port install gdal +curl +expat +geos +hdf4 +hdf5 +netcdf  +openjpeg +postgresql95 +sqlite3
sudo port install gdal +curl +expat +geos +gdal-hdf5 +netcdf  +openjpeg +postgresql95 +sqlite3
#2026/05/28 MacNeo. Error: The '+netcdf' variant has been removed and replaced by the 'gdal-netcdf' subport. I then used
sudo port install gdal +curl +expat +geos +gdal-hdf5   +openjpeg +postgresql95 +sqlite3
export GDAL_DATA=/opt/local/share/gdal
sudo port install scons
sudo port install py313-gdal  #python binding for gdal

sudo port install gmt5 +fftw3 #only for gmt plots
sudo port install py313-rasterio  #for loading and exporting ifgs in python
sudo port install ImageMagick #for exporting interferograms to .kml

##### edit isce2-2.6.4/configuration/sconsConfigFile.py line 38 to
GFORTRANFLAGS = ['-ffixed-line-length-none' ,'-fno-second-underscore',   '-fPIC','-fno-range-check','-fallow-argument-mismatch']
```

Create `/Applicatons/isce/SConfigISCE` file for scons

```
PRJ_SCONS_BUILD   = /Applications/isce/isce2-2.6.4/build/isce
PRJ_SCONS_INSTALL = /Applications/isce/isce2-2.6.4/install/isce


LIBPATH = /opt/local/lib
#the last path in CPP is new for autoRIFT in 2.4.x version
CPPPATH = 
/opt/local/Library/Frameworks/Python.framework/Versions/3.13/include/python3.13 
/opt/local/include /opt/local/include/opencv4  /opt/local/lib/opencv4
/opt/local/Library/Frameworks/Python.framework/Versions/3.13/lib/python3.13/site-packages/numpy/core/include
FORTRANPATH = /opt/local/include
FORTRAN = /opt/local/bin/gfortran
CC = /opt/local/bin/gcc
CXX = /opt/local/bin/g++

#libraries needed for mdx display utility
MOTIFLIBPATH = /opt/local/lib       # path to libXm.dylib
X11LIBPATH = /opt/local/lib         # path to libXt.dylib
MOTIFINCPATH = /opt/local/include   # path to location of the Xm
                                    # directory with various include files (.h)
X11INCPATH = /opt/local/include     # path to location of the X11 directory
                                    # with various include files

# turn off CUDA code on this computer
ENABLE_CUDA = FALSE
```

Now install it in `/Applicatons/isce/isce2-2.6.4`

```
rm -rf config.log .sconfig.dblite .sconf_temp .sconsign.dblite;
SCONS_CONFIG_DIR=/Applications/insar_software/isce scons install  
```

Source it with bash file

```
#!/bin/sh

inp="$1"
echo "Loading ISCE $1"
 
export PYTHONPATH=/Applications/insar_software/isce/isce-$1/install:$PYTHONPATH
export PATH=/Applications/insar_software/isce/isce-$1/install/isce/bin:$PATH
export PATH=/Applications/insar_software/isce/isce-$1/install/isce/applications:$PATH
export ISCE_HOME=/Applications/insar_software/isce/isce-$1/install/isce
 
export PATH=/Applications/insar_software/isce/isce-$1/contrib/stack/stripmapStack:$PATH
#export PATH=/Applications/isce/isce$1/contrib/stack/topsStack:$PATH
  
export GDAL_DATA=/opt/local/share/gdal
```

### Troubleshooting

```
### if cmake fails building gdal
xcode-select --reset
xcode-select --install
sudo port clean cmake
sudo port install cmake
sudo port clean gdal
sudo port install gdal +curl +expat +geos +hdf4 +hdf5 +netcdf  +openjpeg 
+postgresql95 +sqlite3


####################################
### Edit autoRIFT in case scons fails to compile ISCE
/Applications/insar_software/isce/isce2-2.5.3_monterey/contrib/geo_autoRIFT
/autoRIFT/bindings/SConscript
###libList = ['gomp','combinedLib','gdal','opencv_core','opencv_highgui',
'opencv_imgproc']
libList = ['opencv_core','opencv_highgui','opencv_imgproc']
####################################
```

If you need to compile an old version of the software, you need to port some files between versions that have been modified to be compatible with the most recent versions of the GNU compiler.

```
update readOrbitPulse.f

mv isce-2.2.0/components/isceobj/Sensor/src/ALOS_pre_process/readOrbitPulse.f isce-2.2.0/components/isceobj/Sensor/src/ALOS_pre_process/readOrbitPulse.f.orig
cp isce2-2.5.3/components/isceobj/Sensor/src/ALOS_pre_process/readOrbitPulse.f isce-2.2.0/components/isceobj/Sensor/src/ALOS_pre_process/.

update cchz_wave.cpp

mv isce-2.2.0/components/mroipac/correlation/src/cchz_wave.cpp isce-2.2.0/components/mroipac/correlation/src/cchz_wave.cpp.orig
cp isce2-2.5.3/components/mroipac/correlation/src/cchz_wave.cpp isce-2.2.0/components/mroipac/correlation/src/.

update looksmodule.cpp

mv isce-2.2.0/components/mroipac/looks/bindings/looksmodule.cpp isce-2.2.0/components/mroipac/looks/bindings/looksmodule.cpp.orig
cp isce2-2.5.3/components/mroipac/looks/bindings/looksmodule.cpp isce-2.2.0/components/mroipac/looks/bindings/.

Edit isce-2.2.0/components/isceobj/Sensor/bindings/SConscript

Change

libList1 = ['alos','DataAccessor','InterleavedAccessor']

to

libList1 = ['DataAccessor','InterleavedAccessor']

Edit isce-2.2.0/components/stdproc/alosreformat/ALOS_fbd2fbs/bindings/SConscript

Change

libList = ['utilLib','ALOSStd','fftw3f']

to

libList = ['utilLib','fftw3f']
```

[Instructions for MacPorts (Piyush Agram, JPL)](https://github.com/piyushrpt/mojaveSetup)

[Instructions for Homebrew (Jose Uribe, CECS, Chile)](https://github.com/juribeparada/homebrew-isce)

[Instructions for Homebrew and Docker (Scott Henderson, U of Washington)](https://github.com/scottyhq/isce_notes)

Issues with unbinded Python libraries: If the Python libraries are correctly installed with macports, but Python cannot import them because you mixed package managers. This issue might happen if you installed libraries with both conda and macports, but then you removed conda like I did once, resulting in a complete mess. Just run the following line

```
export PYTHONPATH=/opt/local/Library/Frameworks/
Python.framework/Versions/3.6/lib/python3.6/site-packages:$PYTHONPATH
```

## Linux installation

[Instructions from Piyush Agram, JPL](https://github.com/piyushrpt/oldLinuxSetup)

### STEP 1: update Python2.7

ISCE is compiled with scons, a Python2.7 application. You have scons installed if you type

```
which scons
scons -h
```

and the outputs do not display errors If you do not have scons, download miniconda2, update the libraries and install scons

```
/home/fdelgado/miniconda2/bin/conda update --all
/home/fdelgado/miniconda2/bin/conda install scons
```

Now type again the scons commands. If it doesn’t work, close the terminal, open another window and try again. ISCE also supports scons for python3.

### STEP 2: update Python3

ISCE is written in Python3 and uses a lot of its libraries. To install them all you need to install anaconda3 and then type

```
/home/fdelgado/anaconda3/bin/conda config --add channels conda-forge
/home/fdelgado/anaconda3/bin/conda update --all 
/home/fdelgado/anaconda3/bin/conda install gdal 
/home/fdelgado/anaconda3/bin/conda install libgdal 
/home/fdelgado/anaconda3/bin/conda install -c omnia fftw3f=3.3.4
```

If they are correctly installed, you should see no errors. The steps in [github](https://github.com/piyushrpt/condaLinuxSetup/blob/reference/dev/anaconda.md) for installing anaconda are slightly different because they state that you should update the libraries before installing `gdal`. I’ve noticed that sometimes the installation is just fine if you do it the way I posted above. If anaconda failed to install the libraries, delete the folder and reinstall from scratch.

The range split spectrum method for ionospheric correction uses `cython`

```
conda install cython
ln -sf /home/fjd49/anaconda3/bin/cython /home/fjd49/anaconda3/bin/cython3
```

If you don’t have the `cython` soft link, the split spectrum module will not be installed

If you are starting from a raw ubuntu installation, you will need to install a few extra libraries, including the OpenMotif library for the MDX interferogram viewer. To do so, just type in the terminal

```
sudo apt-get install libx11-dev libxm4 libmotif-dev libfftw3f-dev gfortran
```

If your system doesn’t find the `fftw3` library, get it from [http://www.fftw.org](http://www.fftw.org/fftw-3.3.8.tar.gz).

For the split spectrum you also need `openCV2`

```
conda install opencv
```

However, this will downgrade `gdal`, which you must then update with

```
conda update gdal
```

### STEP 3: create the SConfigISCE file

This is the tricky part, that ISCE can actually find all the installed libraries. You need to create a file called `SConfigISCE` in the folder above ISCE which specifies the libraries paths.

### STEP 4: Install ISCE

cd to the ISCE folder and then type in the terminal

```
SCONS_CONFIG_DIR=/home/francisco scons install
```

with `/home/francisco` the path of the SCONS_CONFIG file. The ISCE compilation will output thousands of warnings, they are ok, so don’t be scared. Once you’ve succeed to compile the software, you need to add ISCE to your bash profile. If the installation fails due to missing libraries, type

```
rm -rf config.log .sconfig.dblite .sconf_temp
```

and then restart

### STEP 5: source ISCE

Create a .sh file similar to that of macOS with the software path. Source it to load the software

```
source /home/fdelgado/isce/isce.sh
```

You should see a bunch of outputs with no error messages when you type

```
topsApp.py --steps --help
stripmapApp.py --steps --help
```

### STEP 6: get SRTM access

ISCE uses the SRTM DEM (the best free DEM available for InSAR processing, much better than the ASTER GDEM). You need to get a NASA account at [urs.earthdata.nasa.gov](urs.earthdata.nasa.gov) (free).

First, cd to the home directory. Then, create a file named `.netrc` with the following 3 lines

```
machine urs.earthdata.nasa.gov 
login your_earthdata_login_name 
password your_earthdata_password
```

Change `.netrc` permissions with

```
chmod 600 ~/.netrc
```

## CUDA support for Linux

```
#!/bin/bash
#-------- install isce2 gpu ---------#
check your gpu nvidia drivers:
nvidia-smi
+-----------------------------------------------------------------------------+
| NVIDIA-SMI 515.65.01    Driver Version: 515.65.01    CUDA Version: 11.7 |  |  |
| -------------------------------+----------------------+----------------------+ |  |  |
| GPU  Name        Persistence-M | Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp  Perf  Pwr:Usage/Cap | Memory-Usage | GPU-Util  Compute M. |
|  |  | MIG M. |
| ===============================+======================+====================== |  |  |
| 0  Quadro P400         On | 00000000:65:00.0 Off | N/A |
| 34%   35C    P8    N/A /  N/A | 19MiB /  2048MiB | 0%      Default |
|  |  | N/A |
+-------------------------------+----------------------+----------------------+

+-----------------------------------------------------------------------------+
| Processes: |
| GPU   GI   CI        PID   Type   Process name                  GPU Memory |
| ID   ID                                                   Usage |
| ============================================================================= |
| 0   N/A  N/A      1462      G   /usr/lib/xorg/Xorg                  9MiB |
| 0   N/A  N/A      1535      G   /usr/bin/gnome-shell                4MiB |
+-----------------------------------------------------------------------------+
# and cuda:
which nvcc
/usr/local/cuda-11.7/bin/nvcc

# Sabancaya has a Quadro P400

# create env
conda create --name isce2gpu
conda activate isce2gpu

# install isce2 dependencies
mamba install -c conda-forge -y isce2 --only-deps

# install requirements to build
mamba install -c conda-forge -y gcc_linux-64 gxx_linux-64 gfortran_linux-64 cython scons openmotif-dev cmake opencv

# link for isce
ln -s $CONDA_PREFIX/bin/cython $CONDA_PREFIX/bin/cython3

# clone isce in isce2 folder (starting in ~/apps)
git clone https://github.com/isce-framework/isce2
cd isce2

# build folder in /home/dal344/apps/isce2
mkdir build && cd build

# build
cmake .. \
-DCMAKE_PREFIX_PATH=$CONDA_PREFIX  \
-DCMAKE_CUDA_FLAGS="-arch=sm_61" \
-DCMAKE_INSTALL_PREFIX=$CONDA_PREFIX \
-DPYTHON_INCLUDE_DIR=$(python -c "from distutils.sysconfig import get_python_inc; print(get_python_inc)")  \
-DPYTHON_LIBRARY=$(python -c "import distutils.sysconfig as sysconfig; print(sysconfig.get_config_var('LIBDIR'))")

# warnings are fine, but is something looks bad you can repair the code above, delete everything in the build folder (rm -fr *) and run cmake again

# after you think everything looks fine install with:
make -j 6 # to use multiple threads (6)
make install

# remove packages used to build only
mamba remove -y gcc_linux-64 gxx_linux-64 gfortran_linux-64 cython scons openmotif-dev cmake opencv

# isce is installed in : /home/dal344/miniconda3/envs/isce2gpu/packages/isce

# Add ISCE to PATH in .bashrc
#export ISCE_ROOT=/home/dal344/miniconda3/envs/isce2gpu/packages
#export ISCE_HOME=$ISCE_ROOT/isce
## PATH & PYTHONPATH
#export PATH=$ISCE_HOME:$ISCE_HOME/bin:$ISCE_HOME/applications:$PATH
#export PYTHONPATH=$ISCE_ROOT:$PYTHONPATH


#---- stack processors ----#
# we need to copy them to ISCE_HOME
cp -R ~/apps/isce2/contrib/stack $ISCE_HOME/components/contrib/.

## Add to your path: STACK PROCESSOR (PICK ONE and add to $PATH)
#export ISCE_STACK=$ISCE_HOME/components/contrib/stack/topsStack
#export ISCE_STACK=$ISCE_HOME/components/contrib/stack/stripmapStack
# you can add this or not
#export PATH_ALOSSTACK=$ISCE_HOME/components/contrib/stack/alosStack
```

## Earthscope ISCE2 short courses

[2014 short course](https://www.unavco.org/education/advancing-geodetic-skills/short-courses/2014/isce/isce.html)

[2015 short course](https://www.unavco.org/education/professional-development/short-courses/course-materials/insar/2015-advanced-insar-course-materials/2015-advanced-insar-course-materials.html
)

[2016 short course](https://www.unavco.org/education/advancing-geodetic-skills/short-courses/course-materials/insar/2016-insar-isce-giant-course-materials/2016-insar-isce-giant-course-materials.html
)

[2018 short course](https://www.unavco.org/education/professional-development/short-courses/2018/insar-theory-processing/insar-theory-processing.html
)

[Official ISCE forum with answers from the JPL engineers](http://earthdef.caltech.edu/projects/isce_forum/boards)

## Other software

[TRAIN](https://github.com/dbekaert/TRAIN) (David Bekaert, JPL): tropospheric correction software

[MintPy](https://github.com/insarlab/MintPy) (Yunjun Zhang, Heresh Fattahi, U of Miami/Caltech/JPL): time series software

[LICSBAS](https://github.com/yumorishita/LiCSBAS) (Yu Morishita, GSI Japan): time series software for [LICS](https://comet.nerc.ac.uk/comet-lics-portal/) Sentinel-1 interferograms

[PyAPS, Varres, MPITS](http://www.geologie.ens.fr/~jolivet/Research.html) (Romain Jolivet, ENS Paris)

[InSamp](https://myweb.uiowa.edu/wbarnhart/software.html) (Bill Barnhart, U of Iowa): interferogram downsampler and covariance estimator

[GBIS](https://comet.nerc.ac.uk/gbis/) (Marco Bagnardi, NASA): Geodetic Bayesian Inversion Software

[dMODELS](https://www.sciencedirect.com/science/article/abs/pii/S0377027312003836) (Maurizio Battaglia, U.S. Geological Survey)

[Online Mogi inversion example](https://github.com/scottyhq/cov9) (Scott Henderson, U of Washington)

# Interferometric processing workflows

## `stripmapApp.py`

Stripmap processor with geometric coregistration for zero doppler data (COSMO-SkyMED SLC stripmap and spotlight, RADARSAT-2 SLC, TerraSAR-X SKC stripmap and spotlight, ALOS-2 SLC stripmap). It can also process raw data (ENVISAT, ALOS-1, COSMO-SkyMED) in native Doopler geometry. This is the ISCE equivalent of ROI_PAC’s process_2pass.pl which relied on amplitude cross correlation for SLC and DEM alignment instead of the accurate geometric corregistration of this module.

StripmapApp uses geometry to align the SLCs, instead of polynomials like both process_2pass.pl and insarApp.py. Unlike process_2pass.pl and insarApp.py, stripmapApp.py does not incorporate a filtering step during the resampling step. The resamp module also use the polynomial fit for range offsets to perform a crude spectral range filtering.

Unlike ROI_PAC, the range extension (partial chirp compression) is set to a lower amount in the stripmapApp focusing module. This results in interferograms that are 5-10 km shorter in the range direction compared with those processed from raw data wih ROI_PAC.

[stripmapApp internals by Piyush Agram, JPL.](https://www.unavco.org/education/professional-development/short-courses/course-materials/insar/2016-insar-isce-giant-course-materials/Single_Interferogram_Processing_Geometric.pdf)

[Jupyter Notebook in the 2020 UNAVCO ISCE workshop](https://github.com/isce-framework/isce2-docs/blob/reference/Notebooks/UNAVCO_2020/Stripmap/stripmapApp.ipynb)

[Imaging Geometry and Definitions](https://isce-framework.github.io/isce3/overview_geometry.html)

```
stripmapApp.py --steps --help

stripmapApp.py stripmapapp_input.xml --steps    

stripmapApp.py stripmapapp_input.xml --steps --start=step1 --end=step2    
```

### Input example file

For raw data you can skip the Doppler centroid method and the software will calculate automatically with DOPIQ (useDOPIQ), except for COSMO-SkyMed. the latter and for SLC data you have to set it to useDEFAULT.

**ERS data** requires either PRC (DPAF) or ODR (Delft) orbits. PRC orbits are better .

**Doppler centroid**

For ENVISAT raw and ALOS raw data `stripmapApp.py` uses ROI$\_$PAC’s clutterlock algorithm to automatically estimate the Doppler centroid (DOPIQ) . This algorithm estimates the Doppler centroid as a quadratic polynomial function of range multiplied by the PRF. For CSK raw data and zero Doppler data `stripmapApp.py` will read the Doppler centroid in the image metadata. DOPIQ can also calculate the Doppler centroid of CSK raw data, but the centroid provided by ASI is more accurate because the latter is a sixth order polynomial.

### Steps

The stripmapApp steps are

1.  `startup`:

2.  `preprocess`: extract the SLCs.

3.  `cropraw`: crop the raw images.

4.  `formslc`: focus SLC with range-doppler algorithm, skipped for SLCS zero-doppler data.

5.  `cropslc`: crop the SLCs.

6.  `verifyDEM`: check if external DEM is provided, otherwise it downloads SRTM and convert the elevation from the EGM96 geoid to the WGS84 ellipsoid.

7.  `topo`: project DEM to range-azimuth coordinates using the range Doppler equation and some Newton-Raphson solvers.

8.  `geo2rdr`: calculate range and azimuth offsets with geometric coregistration.

9.  `coarse_resample`: coregister secondary SLC with geometric coregistrationonly.

10. `misregistration`: run ampcor on coarsely registered secondary SLC and estimate the average range and azimuth offset to improve the coregistration.

11. `refined_resample`: coregister again the coarse registered secondary SLC with the average azimuth and range offsets from ampcor.

12. `dense_offsets*`: optional step. Run ampcor on coregistered SLCs to extract range and azimuth offsets. These dense offsets can be used for two things. First, if the ionosphere is strong enough to introduce artificial azimuth offsets due to an azimuthal variation in the ionospheric TEC, then the geometric registration is not enough to align the SLCs. In that case the images need to be aligned using variable range and azimuth offsets like in ROI_PAC. Second, they can be used to calculate horizontal displacements for pixel tracking.

13. `rubber_sheet*`: the dense azimuth offset is addedd to the geometric azimuth offset to improve the SLC coregistration.

14. `fine_resample`: coregister the secondary SLC to the reference using the average offsets. If rubbe_sheet was carried out, include the fine azimuth offset field ampcor to correct for potential azimuth misalignments.

15. `split_range_spectrum*`: calculate upper and lower band interferograms by range filtering.

16. `sub_band_resample*`: interpolate the secondary sub-band SLCs with the geometric coregistration (from `topo`) and constant range and azimuth offset (from `misregistration`).

17. `interferogram`: cross multiply reference with finely registered references SLC. Also remove the flat earth and topographic phase with the range offsets file, and apply looks. The interferogram generation step is a simple cross multiplication of reference and coregistered SLCs. The range offsets also represent the phase that is used for flattening. So all 3 pieces are combined together during cross multiplication.

18. `sub_band_interferogram*`: calculate low and high band interferograms.

19. `filter`: filter the interferogram with the Goldstein power spectrum filter.

20. `filter_low_band*`: filter the low band interferogram.

21. `filter_high_band*`: filter the high band interferogram.

22. `unwrap`: unwrap the phase with icu, grass, snaphu, snaphu_mcf.

23. `unwrap_low_band*`: unwrap the filtered low band interferogram.

24. `unwrap_high_band*`: unwrap the filtered high band interferogram.

25. `ionosphere*`: calculate the ionospheric correction with formula 7 of .

26. `geocode`: geoode all prodcuts.

27. `geocodeoffsets`: optional step for pixel tracking only, geocode range and azimuth offsets.

28. `endup`

\*Optional steps for ionospheric correction with the split spectrum method.

You can output the SAR sampling parameters of each image with the following

    grep -n -e number_of_lines -e number_of_samples stripmapProc.xml

    grep -n wavelength stripmapProc.xml
    grep -n lookSide stripmapProc.xml #-1 right looking almost all civilian missions
    grep -n polarization stripmapProc.xml
    grep -n antenna_length stripmapProc.xml ##ESA's missions VV, others might use HH

    grep -n -e sensing_start -e sensing_mid -e sensing_stop stripmapProc.xml

    grep -n range_sampling_rate stripmapProc.xml
    grep -n range_pixel_size stripmapProc.xml
    grep -n -e  starting_range -e far_range stripmapProc.xml
    grep -n prf stripmapProc.xml
    grep -n incidence_angle stripmapProc.xml
    grep -n squint_angle stripmapProc.xml
    grep -n doppler_vs_pixel stripmapProc.xml

### Process an unflattened interferogram

stripmapApp uses the range offset file to automatically flatten the interferogram – unlike ROI_PAC that implemented the flattening and simulation removal in different runs of the old `diffnsim.f` routine. You can use the following csh script to calculate an interferogram before the flat earth and simulation phase corrections.

```
#!/bin/csh
###calculate unflattened interferogram for ISCE stripmapApp.py

#extract radar wavelength
set wvl = `grep wavelength stripmapProc.xml | head -1 | sed 's/<wavelength>//g' | sed 's/<\/wavelength>//g'`

#extract slant range pixel size
set rps = `grep range_pixel_size stripmapProc.xml | head -1 | sed 's/<range_pixel_size>//g' | sed 's/<\/range_pixel_size>//g'`

#calculate interferogram and add back topo+flat earth phase
imageMath.py -e="a*conj(b)*exp(J*4*PI*${rps}*c/${wvl})"  --a=reference_slc/reference.slc  --b=coregisteredSlc/refined_coreg.slc --c=offsets/range.off -t CFLOAT -o unflat.int

looks.py -i unflat.int -a ${alks} -r ${rlks}
```

Here `rps` is the range pixel size to convert the range offsets to meters and `wvl` is the radar wavelength.

### Dense offsets

ISCE can calculate range and azimuth offsets on the coregistered SLCs. These are the same offset fields that are used to coregister the SLCs, but here they also represent the horizontal displacement field. Their accuracy is much lower compared to that of InSAR, usually 1/10 of the pixel size. Hence for SAR pixel sizes of 2-20 m, pixel offsets have accuracies of 0.2-2 m. Hence, they are only useful for very large events. Pixel offsets are particularly useful for large continental earthquakes with surface ruptures and $M_W$ 6.5 - 7.9. The large strain near the surface rupture results in coherence loss in most of the interferograms unless either the pixel size is very small or the radar operates with L-band. However, pixel offsets from SAR images can retrieve the deformation in those areas (e.g., ), and are not subject to either aliasing or phase unwrapping problems.

You can use the following set of parameters in the input file of `stripmapApp.py`.

```
<property name="do denseoffsets">True</property>
<property name="dense window width">32</property> <!-- 64 default -->
<property name="dense window height">32</property> <!-- 64 default -->
<property name="dense skip width">16</property> <!-- 32 default -->
<property name="dense skip height">16</property> <!-- 32 default -->
<property name="dense search width">20</property> <!-- 20 default-->
<property name="dense search height">20</property> <!-- 20 default-->
<property name="offset geocode list">["denseOffsets/denseOffsets.bil",
"denseOffsets/denseOffsets_snr.bil"]</property>
```

These parameters work very well for running offsets on CSK stripmap and ALOS-2 SM1 data (2 m/pixel). The parameters to change are the dense window width/height that controls the quality of the offset field and the dense skip width/height that controls the pixel size. A smaller dense skip implies a smaller pixel size but reducing the skip will increase the processing time. The window size is set by trial and error. If you want to run offsets on data sets with non square pixels like ALOS-2 SM3 and Sentinel-1 TOPS (pixel ratios of 2 and 1/4 respectively) you need to take into account the pixel ratio in both the window and skip size. This is an analog to take looks.

Pixel offsets are not very useful for volcanic applications unless the displacement field is larger than 1 m resulting in a coherence loss because the strain field exceeds the maximum strain threshold of the data set . Therefore, if your interferogram is decorrelated due to large deformation, you can still extract useful deformation using pixel tracking. Events of these kind are usually dike intrusions at basaltic calderas like Dabbahu 2005 , Sierra Negra 2005 and 2018 (, Shreve and Delgado, under review), Kilauea 2007 , Nabro 2011 , Hohluraun 2014 , Kilauea 2018, Ambrym 2015 and 2018 , Taal 2020 , and Mauna Loa 2022. On the other hand, if your deformation signal is smaller, like 0.05-0.5 m as found in almost all of the deforming volcanoes on Earth, then the deformation is too small to be accurately measured by pixel tracking, so you’re better off working with InSAR data only. For these data sets it is better to use a data set with a small pixel size like that of ALOS-2 SM1 or X-band stripmap (2 m/pixel). In my experience, there is no need to use pixel offsets for volcanic applications unless you study one of these huge deformation events.

### Directories

<div id="tab:smapp">

| **File** | **Description** |
| --- | --- |
| stripmapProc.xml | Metadata file |
| reference_slc.xml | Metadata file |
| reference_slc | reference SLC image |
| references_slc | references SLC image |
| references_slc.xml | Metadata file |
| geometry/los.rdr | Look and heading angles of every pixel |
| geometry/lat.rdr | Latitude of every pixel |
| geometry/lon.rdr | Longitude of every pixel |
| geometry/hgt.rdr | Elevation of every pixel |
| coregisteredSlc | Coarse and fine aligned secondary SLCs |
| offsets | Range and azimuth offsets. In geometric coregistration workflows the range offset file contains both the flat earth and the simulation phase |
| misreg | Average range and azimuth offset calculated by ampcor |
| interferogram/topophase.flat | Flattened differential interferogram. |
| interferogram/topophase.cor | Coherence from the SLC’s |
| interferogram/filt_topophase.flat | Filtered interferogram |
| interferogram/phsig.cor | Effective coherence based on the local phase variance or phase sigma, higher than regular coherence |
| interferogram/filt_topophase.unw | Unwrapped filtered interferogram |
| interferogram/filt_topophase.flat.geo | Geocoded filtered interferogram |
| interferogram/filt_topophase.unw.geo | Geocoded unwrapped interferogram |
| dem.crop | Topography of the geocoded interferogram, elevation with respect to the WGS84 ellipsoid |

Output files of `stripmapApp.py`.

</div>

## topsApp.py

TOPS processor with geometric coregistration for Sentinel-1 IW data. The SLC focusing is detailed in and the [ESA Sentinel-1 User Guide](https://sentinel.esa.int/web/sentinel/technical-guides/sentinel-1-sar/products-algorithms/level-1-algorithms/overview).

[Jupyter Notebook in the 2020 UNAVCO ISCE workshop](https://github.com/isce-framework/isce2-docs/blob/reference/Notebooks/UNAVCO_2020/TOPS/topsApp.ipynb)

[topsApp internals by ISCE team, April 2016, first version of this processor.](https://imaging.unavco.org/software/ISCE/topsApp_ISCE_20160418.pdf)

[topsApp internals by Heresh Fattahi, Piyush Agram and Mark Simons, JPL.](https://files.scec.org/s3fs-public/0129_1400_1530_Fattahi.pdf)

```
topsApp.py --steps --help

topsApp.py topspapp_input.xml --steps    

topsApp.py topspapp_input.xml --steps --start=step1 --end=step2    
```

### Steps

Here I only describe the steps that are different than those in stripmapApp.py

1.  `startup `

2.  `preprocess `

3.  `computeBaselines `

4.  `verifyDEM `

5.  `topo`: same as stripmapApp but for each individual burst

6.  `subsetoverlaps`: extract the burst overlap regions

7.  `coarseoffsets`: calculate coarse offsets with ampcor in the burst overlap regions for the upper and lower bursts

8.  `coarseresamp`: interpolate the secondary burst SLC to the reference one with the coaarse offset field

9.  `overlapifg`: calculate differential and upper and lower interferograms

10. `prepesd`: calculate frequency difference for ESD

11. `esd`: enhanced spectral diversity correction for the accurate azimuth misregistration in the burst overlap regions

12. `rangecoreg`: calculate range offsets only with ampcor for fine coregistration

13. `fineoffsets`: calculate geometric offsets for every pair of bursts

14. `fineresamp`: interpolate the secondary burst SLC to the reference one with the geometric offset, fine range offsets and ESD

15. `ion`: calculate ionospheric correction for burst overlap regions. This might be necessary if you work in a low geomagnetic latitude and if you observe phase discontinuities in the burst overlap regions after accounting for ESD.

16. `burstifg`: calculate interferograms for every bursts

17. `mergebursts`: merge the burst interferograms

18. `filter `

19. `unwrap `

20. `unwrap2stage`: use a linear programming solver to solve for the constants between the connected componets file of the phase unwrappoint output

21. `geocode`

22. `denseoffsets `

23. `filteroffsets `

24. `geocodeoffsets`

### Directories

<div id="tab:topsapp">

| **File** | **Description** |
| --- | --- |
| reference | reference SLC image |
| references | references SLC image |
| geom$\_$reference/los.rdr | Look and heading angles of every pixel |
| geom$\_$reference/lat.rdr | Latitude of every pixel |
| geom$\_$reference/lon.rdr | Longitude of every pixel |
| geom$\_$reference/hgt.rdr | Elevation of every pixel |
| coarse$\_$offsets | Range and azimuth offsets for the forward and backward SLCs in the burst overlap regions |
| coarse$\_$coreg | Coregistered forward and backward references SLCs in the burst overlap regions |
| coarse$\_$interferogram | Forward and backward references interferograms in the burst overlap regions calculated for the ESD correction |
| ESD | enhanced spectral diversity correction to minimize azimuth misregisrtation problems due to the azimuthal variation in the Doppler centroid |
| fine$\_$offsets | Burst by burst range and azimuth offsets |
| fine$\_$coreg | Burst by burst references SLC after coregistration |
| fine$\_$interferogram | Burst by burst coherence and interferogram after topographic and ESD correction |
| merged/topophase.flat | Differential interferogram with topography removed |
| merged/topophase.cor | Coherence from the SLC’s |
| merged/filt_topophase.flat | Filtered interferogram |
| merged/phsig.cor | Effective coherence based on the local phase variance or phase sigma, higher than regular coherence |
| merged/filt_topophase.unw | Unwrapped filtered interferogram |
| merged/filt_topophase.flat.geo | Geocoded filtered interferogram |
| merged/filt_topophase.unw.geo | Geocoded unwrapped interferogram |
| dem.crop | Topography of the geocoded interferogram, elevation with respect to the WGS84 ellipsoid |

Output files of `topsApp.py`. All the products are extracted into subfolders called IW$_{i}$, with i the swath number from 1 to 3.

</div>

## alos2App.py

### Steps

The interferogram is calculated in three steps. First, subtract the phase. Second remove the flat earth. Third, remove the simulation. This specific part of the workflow is more akin to process_2pass.pl in ROI_PAC than to stripmapApp.py

1.  `startup`

2.  `preprocess`

3.  `baseline`

4.  `download_dem`

5.  `prep_slc`

6.  `slc_offset`: coregister SLCs with ampcor. Unlike `stripmapApp.py`, `alos2App.py` doesn’t use geometry to align the SLC’s.

7.  `form_int`: calculate interferogram

8.  `swath_offset`: for ScanSAR only

9.  `swath_mosaic`: for ScanSAR only

10. `frame_offset`: for ScanSAR only

11. `frame_mosaic`: for ScanSAR only

12. `rdr2geo`: calculate geometry and simulation in radar coordinates for multilooked interferogram. This is similar to the topo step in `stripmapApp.py` and `topsApp.py`

13. `geo2rdr`: calculate synthetic phase from geometry and simulation

14. `rdrdem_offset`: cross correlate DEM with flattened interferogram

15. `rect_rgoffset`: rectify range offset file to reference SLC.

16. `diff_int`: calculate differential interferogram

17. `look`

18. `coherence`

19. `ion_subband`: split the range spectrum in two bands from the carrier frequency. This result in a low band and an high band interferogram

20. `ion_unwrap`: unwrap sub-band interferograms

21. `ion_filt`: filter unwrapped sub-band interferograms

22. `ion_correct`: decompose the phase into dispersive and non-dispersive components from the unwrapped sub-band interferograms and remove dispersive component from the differential interferogram

23. `filt`

24. `unwrap`

25. `geocode`

26. `slc_mosaic`

27. `slc_match`

28. `dense_offset`

29. `filt_offset`

30. `geocode_offset`

## Miscellaneous issues

### Multilooking

The resolution of the DEM and the SLCs never agree with each other, but the interfeorgrams will be geocoded to the same posting of the DEM. To reduce the amount of interpolation due to the resolution mismatch, one usually takes the number of looks closer to the native resolution of the DEM and it is fine to accept some interpolation (`tab:slcres` and `tab:slcres_dem`). For example, the native resolution of the ALOS-1 FBS data is 10 m/pixel and we usually apply 4 looks to reduce the speckle noise and to average the images to a posting (40 m/pixel) close to the resolution of the SRTM DEM (30 m/pixel). Sometimes it is necessary to apply additional looks, and significantly reduce the size of the interferogram with respect to the input DEM in order to increase the signal to noise ratio. If this happens, you might have to apply additional looks to the geocoded interferogram because it might look oversampled with too much interpolation.

### Filtering

The default filtering in ISCE is a power spectrum filtering coefficient of 0.5 . The result is a strong filter, and as a rule of thumb, I use between 0.3 and 0.7. Higher strengths are only recommended if the data is strongly decorrelated because a strong filtering can distort significantly the interferogram, especially when you have many fringes that are very close to each other. You should always check whether the filtering introduces artifacts or distorts the shape of the fringes or not.

### Phase Unwrapping

ISCE includes four unwrapping algorithms: `grass`, `icu` (Integrated Correlation and Unwrapping), `snaphu` and `snaphu_mcf`. The first three are variations of the residue-cut method of and are based on two ideas. First unwrapping is carried out integrating the phase differences in a residue (noise) free trajectory in order to avoid phase jumps due to noise. Second, under the assumption that the phase varies less than one half phase cycle ($\pi$ rads), which means that the deformation is smooth. This way the unwrapping result is independent of the integration trajectory. `snaphu` (Statistical Network Approach to Phase Unwrapping) is an improved branch and cut algorithm. `snaphu_mcf` (Statistical Network Approach to Phase Unwrapping - Minimum Cost Flow) works very differently because it is a combination of a probability density function and an optimization network flow algorithm and will unwrap the whole image . `icu` works well with interferograms with few fringes and smooth signals (such as the ALOS-1 Kilauea and Pichilemu interferograms), while `snaphu_mcf` deals much better with decorrelated interferograms, decorrelated areas and many fringes. If you use `snaphu_mcf` to unwrap, the file can be masked with the `maskunw.py` Python script which uses both the coherence and the connected components file. Remember that no unwrapping algorithm is perfect and you should always manually check the quality of the unwrapping and the filtering.

### Color scale

For an interferogram processed with the most recent image as the reference, **a concentric fringe from red to blue to green to yellow is mostly subsidence or line-of-sight increase**. If the color pattern is reversed, then it is uplift.

If you want to reprocess the interferogram with different number of looks, do not remove the .slc files. Do not remove the reference/reference directories because they contain the SLCs metadata.

### More on taking looks

If you need to reduce the size of a file, you can take 3 looks with

```
looks.py -i infile.geo -a 3 -r 3
```

For example, if you set the posting to 90 m/pixel but the interferogram is processed with the 30 m SRTM, the final interferogram will be geocoded at the same resolution of the DEM. In this case there is a clear oversampling issue as the output product has a much higher resolution than the unwrapped int in radar coordinates. If this is the case, you should decrease the size of the final geocoded interferogram from  30 to  90 m/pixel with

```
looks.py -i filt_topophase.unw.geo -a 3 -r 3
```

`topsApp.py` includes the option to geocode with a lower resolution DEM, like we did for the Calbuco eruption and the Chiloe earthquake. You only have to provide a lower resolution DEM with

```
<property name="geocode demFilename">~/insarproc/dem/srtm_svz90m.dem</property>
```

To open an interferogram,

```
mdx.py topophase.flat topophase.unw
```

If you need to change the wrapping rate, right click on Unw, then Scale Mode -$>$ PER -$>$ Wrap and change the number to an arbitrary number that is not a multiple of 2$\pi$. You will have to do the same for the wrapped interferogram, otherwise the displayed color scale is wrong. To change the amplitude color scale, click on amp, then on Scale Mode -$>$ PER and change it. The real numbers after the COL and ROW are the pixel values of the interferogram bands and are in the same order than the opened files. This is useful to verify that the phase was correctly unwrapped.

### Heading and look angle file

Historically, people did not all use the same sign conventions in InSAR. Check whether your interferograms are ‘range change’ or ‘ground LOS displacement’. Check if your pointing vectors are consistent with your interferograms (pointing from satellite to ground, or ground to satellite?).

ISCE does not output heading (azimuth) direction in a geographical convention. It uses a right-hand rule from east (i.e. 0=east, and counts positive degrees counter-clockwise from there). To convert to geographical heading, multiply the azimuth by -1 and add 90$º$.

[Math formula to generate the directional cosines from the LOS file directly with the software](http://earthdef.caltech.edu/boards/4/topics/327). This is equivalent to formulas implemented in the `load\_isce.m` scripts.

```
imageMath.py --eval='sin(rad(a_0))*cos(rad(a_1+90));  sin(rad(a_0)) * sin(rad(a_1+90)); cos(rad(a_0))'  --a=los.rdr.geo -t FLOAT -s BIL -o enu.rdr.geo
```

$$S_{1} = cos(h+90)sin(l); S_{2} = sin(h+90)sin(l); S_{3} = cos(l)$$ Here $l$ is the look angle and $h$ is the satellite heading, $S_i$ are the directional cosines and the line-of-sight displacement $U_{LOS}$ is

$$U_{LOS} = \vec{S} \bullet \vec{U} = S_{1}U_{x} + S_{2}U_{y} + S_{3}U_{z}$$ with $U_{i}$ the displacement in the $_{i}$ direction. For pressure sources like those in volcanoes most the displacement projected into the LOS is vertical so it is standard practice to describe those signals as either "uplift" or "subsidence", even though it is incorrect from the line-of-sight point of view. For earthquakes both the east-west and vertical components can have equal magnitudes, so in the case it is more accurate to talk about line-of-sight displacement increase/decrease.

# Ancillary information required for InSAR processing

## DEMs

If your area of interest does not require high resolution processing and accurate DEM (better than 30 m/pixel and accuracies better than 10 m), then use SRTM. Do not use the ASTER GDEM because it is less accurate than SRTM ($\sim$20 vs $\sim$10 m respectively), unless you are outside of +/- 60º of latitude (e.g., Iceland). Ideally try to use DEMs with resolution similar to the images resolution. TanDEM-X DEM is better than SRTM but only free for 90 m/pixel. TanDEM-X 1 and 0.4 arcsec DEMs and the optical Pléiades data require a DEM proposal to be submitted to DLR and CNES respectively. The Pléiades DEMs have a native resolution of 0.5 m/pixel and are downsampled to 2 m/pixel to reduce the effect of noise. Rarely a SAR data set is processed to such high resolution for tectonic and volcanic applications. Therefore the Pléaides DEMs are usually downsampled down to 10 m/pixel (`tab:slcres_dem`).

If the USGS server is down you can fetch SRTM from the ESA repository

```
dem.py -a stitch -b 38 39 -112 -111 -r -s 1 -c -u http://step.esa.int/auxdata/dem/SRTMGL1/
```

The Copernicus DEM from ESA and AIRBUS is a masked and corrected version of the TanDEM-X 30/90 m DEMs. It is the most up to date, high resolution and high accuracy DEM to date. You can get it from [PANDA](http://panda.copernicus.eu) for free. Be sure to download the DTE product (elevation in integer numbers). After you download the tiles you need to merge them, change the vertical datum from the EGM2008 geoid to the WGS84 ellipsoid, and then convert it to an i2 integer file format that ISCE can read. This can be easily done with GDAL

```
ls *tar | awk '{print "tar -xf",$1}' > t.csh ; csh t.csh

gdal_merge.py-3.7 Copernicus_DSM*/DEM/* -o cop_dem_glo30_svz.tif

gdalwarp -s_srs "+proj=longlat +datum=WGS84 +no_defs  +geoidgrids=egm08_25.gtx" -t_srs "+proj=longlat  +ellps=WGS84 +datum=WGS84 +no_defs"  cop_dem_glo30_svz.tif cop_dem_glo30_svz_wgs84.tif

gdal_translate cop_dem_glo30_svz_wgs84.tif -Of ISCE cop_dem_glo30_svz_wgs84.dem

mdx.py cop_dem_glo30_svz_wgs84.dem -z -10
```

## ENVISAT and Sentinel-1A/B processing

[ASAR overview](https://earth.esa.int/eogateway/instruments/asar/description)

[ERS / ENVISAT orbits](http://topex.ucsd.edu/gmtsar/tar/ORBIts.tar)

[ENVISAT instrument files](https://earth.esa.int/web/sppa/mission-performance/esa-missions/envisat/asar/products-and-algorithms/products-information/aux)

[Sentinel-1A/B restituted orbits. ](https://s1qc.asf.alaska.edu/aux_resorb/)They are available hours after the SLC acquisition

[Sentinel-1A/B final precise orbits.](https://s1qc.asf.alaska.edu/aux_poeorb/) They are available $\sim$3 weeks after the SAR image acquisition.

[Sentinel-1A/B CAL instrument files, required for the elevation antenna pattern correction [use id=3 according to this](http://earthdef.caltech.edu/boards/4/topics/1955)](https://qc.sentinel1.eo.esa.int/aux_cal/?instrument_configuration_id=3)

The Sentinel-1 interferograms can be processed with either the header, restituted and final orbit. The header orbit is very inaccurate (only one state vector at the beginning and the end of the acquisition). These preliminary orbits are fine for quick results, like a quick co-eruptive or co-seismic interferogram, but for accurate interferograms you should always use the precise orbit. External orbits are not required for other data sets. However, RADARSAT-2 orbits are of lower quality due to the smaller amount of state vectors provided by CSA. This implies that RSAT-2 data can often have large long wavelength ramps that look alike those in ENVISAT interferograms processed with the old ROI_PAC software. CSK intererograms tend to have long wavelength ramps compared with those from TSX. Precise orbits are available for ALOS-2 starting $\sim$3 days after image acquisition.

# Clean-up directories

These commands will only leave the multilooked interferograms and relevant files. They will remove the full resolution intermediate products.

`stripmapApp.py`

```
rm -rfv */*.slc */*.raw offsets/*.off geometry/*.full interferogram/*.full
```

`topsApp.py`

```
rm -v fine_interferogram/IW?/burst_??.int fine_interferogram/IW?/burst_??.cor fine_offsets/IW?/azimuth*off fine_offsets/IW?/range*off fine_coreg/IW?/burst_??.slc geom_reference/IW?/???_??.rdr ESD/overlap_IW?_??.int coarse_coreg/overlaps/IW?/*slc coarse_offsets/overlaps/IW?/*.off coarse_interferogram/overlaps/IW?/burst_bot_??_??.int coarse_interferogram/overlaps/IW?/burst_top_??_??.int */*full
```

`alos2App.py` for ScanSAR

```
rm -v  f?_????/s?/*.slc f?_????/s?/*.int  f?_????/s?/*.amp  f?_????/mosaic/*.int  f?_????/mosaic/*.amp  ion/*/f?_????/s?/*.int  ion/*/f?_????/s?/*.amp  ion/*/insar/*.int  ion/*/insar/*.amp  ion/*/*/mosaic/*.amp ion/*/*/mosaic/*.int insar/*_1rlks_14alks.???  insar/*_1rlks_14alks_??.??? insar/*_1rlks_14alks_rg_rect.off  insar/rdr_dem_offset/*_1rlks_14alks.??? insar/rdr_dem_offset/???_3rlks_14alks.float
```

TOPS stack processor

```
rm -v interferograms/2*/IW?/fine_??.int coreg_secondarys/2*/IW?/*off coreg_secondarys/2*/IW?/*slc geom_reference/IW?/???_??.rdr  geom_reference/IW?/shadow*.rdr geom_reference/IW?/incLocal*.rdr coreg_secondarys/2*/overlap/IW?/*off coreg_secondarys/2*/overlap/IW?/*slc ESD/20*/IW?/overlap_*.int coarse_interferograms/20*/overlap/IW?/*int
```

Remove only the intermediate files (bottom and top burst overlap interferograms, and double difference interferogram for ESD). `topsApp.py`

```
rm -v ESD/*/IW?/freq_??.bin ESD/*/IW?/overlap_??.int
rm -v coarse_interferograms/*/overlap/IW?/*int
```

# Example interferograms of volcanoes and earthquakes from ENVISAT, ALOS, Sentinel-1, and ALOS-2

## Volcanoes

### Okmok, July 2008 eruption, ENVISAT

<figure>
  <img src="figures/okmok_envisat_20080716_20071010.png" alt="okmok envisat 20080716 20071010" width="900">
  <figcaption><strong>Figure.</strong> 2008 Okmok eruption, ENVISAT 2008/07/16 - 2007/10/10.</figcaption>
</figure>


ENVISAT C-band ascending interferogram

<https://imaging.unavco.org/data/sar/lts/winsar/ENV1/451/1071/ASA_IM__0CNPDK20080627_084349_000000172069_00451_33070_6667.N1>

<https://imaging.unavco.org/data/sar/lts/winsar/ENV1/451/1071/ASA_IM__0CNPDK20080801_084350_000000162070_00451_33571_6940.N1>

<https://imaging.unavco.org/data/sar/lts/winsar/ENV1/222/1071/ASA_IM__0CNPDK20071010_084642_000000172062_00222_29334_4163.N1>

<https://imaging.unavco.org/data/sar/lts/winsar/ENV1/222/1071/ASA_IM__0CNPDK20080716_084642_000000172070_00222_33342_6798.N1>

### Kilauea, June 2007 Father’s Day dike intrusion, ALOS

ALOS-1 L-band ascending

<https://datapool.asf.alaska.edu/L1.0/A3/ALPSRP068020370-L1.0.zip>

<https://datapool.asf.alaska.edu/L1.0/A3/ALPSRP074730370-L1.0.zip>

ALOS-1 L-band descending

<https://datapool.asf.alaska.edu/L1.0/A3/ALPSRP058463230-L1.0.zip>

<https://datapool.asf.alaska.edu/L1.0/A3/ALPSRP078593230-L1.0.zip>

### Kilauea 2018 dike intrusion and caldera collapse, Sentinel-1

Sentinel-1 C-band ascending interferograms

<https://datapool.asf.alaska.edu/SLC/SA/S1A_IW_SLC__1SDV_20180420T043026_20180420T043054_021546_025211_81BE.zip>

<https://datapool.asf.alaska.edu/SLC/SA/S1A_IW_SLC__1SDV_20180502T043026_20180502T043054_021721_025793_5C18.zip>

<https://datapool.asf.alaska.edu/SLC/SB/S1B_IW_SLC__1SDV_20180508T042948_20180508T043023_010825_013CA6_6EC4.zip>

<https://datapool.asf.alaska.edu/SLC/SA/S1A_IW_SLC__1SDV_20180514T043027_20180514T043055_021896_025D31_C20C.zip>

Sentinel-1 C-band descending interferograms

<https://datapool.asf.alaska.edu/SLC/SB/S1B_IW_SLC__1SDV_20180423T161524_20180423T161551_010613_0135DD_33F0.zip>

<https://datapool.asf.alaska.edu/SLC/SB/S1B_IW_SLC__1SDV_20180505T161525_20180505T161552_010788_013B7D_25D2.zip>

<https://datapool.asf.alaska.edu/SLC/SA/S1A_IW_SLC__1SDV_20180511T161552_20180511T161622_021859_025BF8_1292.zip>

<https://datapool.asf.alaska.edu/SLC/SA/S1A_IW_SLC__1SDV_20180511T161620_20180511T161639_021859_025BF8_9FF0.zip>

<https://datapool.asf.alaska.edu/SLC/SB/S1B_IW_SLC__1SDV_20180517T161525_20180517T161552_010963_01411F_59D5.zip>

[Processed interferograms](http://pgf.soest.hawaii.edu/Kilauea_insar/)

<figure>
  <img src="figures/kilauea_s1_20180511_20180505.png" alt="kilauea s1 20180511 20180505" width="900">
  <figcaption><strong>Figure.</strong> 2018 Kilauea caldera collapse and east rift zone deflation, Sentinel-1 descending 2018/05/11 - 2018/05/05.</figcaption>
</figure>

## Earthquakes

### June 2015 M$_W$ 6.3 Pishan, thrust faulting, Sentinel-1

Sentinel-1 C-band ascending (2 frames)

<https://datapool.asf.alaska.edu/SLC/SA/S1A_IW_SLC__1SSV_20150724T124030_20150724T124058_006953_0096A6_64D5.zip>

<https://datapool.asf.alaska.edu/SLC/SA/S1A_IW_SLC__1SSV_20150724T124056_20150724T124123_006953_0096A6_544B.zip>

<https://datapool.asf.alaska.edu/SLC/SA/S1A_IW_SLC__1SSV_20150630T124029_20150630T124057_006603_008CD2_C445.zip>

<https://datapool.asf.alaska.edu/SLC/SA/S1A_IW_SLC__1SSV_20150630T124055_20150630T124122_006603_008CD2_7965.zip>

<figure>
  <img src="figures/s1_asc_20150724_20150630.png" alt="s1 asc 20150724 20150630" width="900">
  <figcaption><strong>Figure.</strong> 2015 Pishan earthquake, thrust faulting, Sentinel-1 ascending 2015/07/24 - 2015/06/30.</figcaption>
</figure>


Sentinel-1 C-band descending (2 frames)

<https://datapool.asf.alaska.edu/SLC/SA/S1A_IW_SLC__1SSV_20150718T004905_20150718T004933_006858_0093F8_5FF1.zip>

<https://datapool.asf.alaska.edu/SLC/SA/S1A_IW_SLC__1SSV_20150718T004931_20150718T004958_006858_0093F8_B2A4.zip>

<https://datapool.asf.alaska.edu/SLC/SA/S1A_IW_SLC__1SSV_20150624T004905_20150624T004932_006508_008A3D_3A0D.zip>

<https://datapool.asf.alaska.edu/SLC/SA/S1A_IW_SLC__1SSV_20150624T004930_20150624T004957_006508_008A3D_BE38.zip>

### April 2015 M$_W$ 7.8 Gorkha, thrust faulting

[ALOS-2 interferograms](https://topex.ucsd.edu/nepal/)

### November 2016 M$_W$ 7.8 Kaikoura, strike-slip faulting

[ALOS-2 interferograms](https://topex.ucsd.edu/NZ_EQ/)

### July 2019 M$_W$ 7.1 Ridgecrest, strike-slip faulting

[ALOS-2 and Sentinel-1 interferograms](https://topex.ucsd.edu/SV_7.1/)

### Feb 2023 doublet M$_W$ 7.8, 7.5 Turkey, strike-slip faulting, ALOS-2

<figure>
  <img src="figures/filt_220916-230217_5rlks_28alks_msk.png" alt="filt 220916-230217 5rlks 28alks msk" width="900">
  <figcaption><strong>Figure.</strong> 2023 Turkey doublet, strike-slip faulting, ALOS-2 ScanSAR descending 2022/09/16 - 2023/02/17. The area is 350$×$700 km$^2$ (2 frames, 5 swaths).</figcaption>
</figure>


<https://www.eorc.jaxa.jp/ALOS/en/dataset/alos_open_and_free_e.htm>

Frame 2850, 2022/09/16

<https://jaxaalos2.s3.us-west-2.amazonaws.com/palsar2-scansar/Turkey-Syria-earthquake/L1.1/BRS-HH-ALOS2449082850-220916-WBSR1.1__D-F1.jpg>

<https://jaxaalos2.s3.us-west-2.amazonaws.com/palsar2-scansar/Turkey-Syria-earthquake/L1.1/BRS-HH-ALOS2449082850-220916-WBSR1.1__D-F2.jpg>

<https://jaxaalos2.s3.us-west-2.amazonaws.com/palsar2-scansar/Turkey-Syria-earthquake/L1.1/BRS-HH-ALOS2449082850-220916-WBSR1.1__D-F3.jpg>

<https://jaxaalos2.s3.us-west-2.amazonaws.com/palsar2-scansar/Turkey-Syria-earthquake/L1.1/BRS-HH-ALOS2449082850-220916-WBSR1.1__D-F4.jpg>

<https://jaxaalos2.s3.us-west-2.amazonaws.com/palsar2-scansar/Turkey-Syria-earthquake/L1.1/BRS-HH-ALOS2449082850-220916-WBSR1.1__D-F5.jpg>

<https://jaxaalos2.s3.us-west-2.amazonaws.com/palsar2-scansar/Turkey-Syria-earthquake/L1.1/IMG-HH-ALOS2449082850-220916-WBSR1.1__D-F1>

<https://jaxaalos2.s3.us-west-2.amazonaws.com/palsar2-scansar/Turkey-Syria-earthquake/L1.1/IMG-HH-ALOS2449082850-220916-WBSR1.1__D-F2>

<https://jaxaalos2.s3.us-west-2.amazonaws.com/palsar2-scansar/Turkey-Syria-earthquake/L1.1/IMG-HH-ALOS2449082850-220916-WBSR1.1__D-F3>

<https://jaxaalos2.s3.us-west-2.amazonaws.com/palsar2-scansar/Turkey-Syria-earthquake/L1.1/IMG-HH-ALOS2449082850-220916-WBSR1.1__D-F4>

<https://jaxaalos2.s3.us-west-2.amazonaws.com/palsar2-scansar/Turkey-Syria-earthquake/L1.1/IMG-HH-ALOS2449082850-220916-WBSR1.1__D-F5>

<https://jaxaalos2.s3.us-west-2.amazonaws.com/palsar2-scansar/Turkey-Syria-earthquake/L1.1/LED-ALOS2449082850-220916-WBSR1.1__D>

<https://jaxaalos2.s3.us-west-2.amazonaws.com/palsar2-scansar/Turkey-Syria-earthquake/L1.1/summary_ALOS2449082850-220916-WBSR1.1__D.txt>

<https://jaxaalos2.s3.us-west-2.amazonaws.com/palsar2-scansar/Turkey-Syria-earthquake/L1.1/TRL-ALOS2449082850-220916-WBSR1.1__D>

<https://jaxaalos2.s3.us-west-2.amazonaws.com/palsar2-scansar/Turkey-Syria-earthquake/L1.1/VOL-ALOS2449082850-220916-WBSR1.1__D>

Frame 2900, 2022/09/16

<https://jaxaalos2.s3.us-west-2.amazonaws.com/palsar2-scansar/Turkey-Syria-earthquake/L1.1/BRS-HH-ALOS2449082900-220916-WBSR1.1__D-F1.jpg>

<https://jaxaalos2.s3.us-west-2.amazonaws.com/palsar2-scansar/Turkey-Syria-earthquake/L1.1/BRS-HH-ALOS2449082900-220916-WBSR1.1__D-F2.jpg>

<https://jaxaalos2.s3.us-west-2.amazonaws.com/palsar2-scansar/Turkey-Syria-earthquake/L1.1/BRS-HH-ALOS2449082900-220916-WBSR1.1__D-F3.jpg>

<https://jaxaalos2.s3.us-west-2.amazonaws.com/palsar2-scansar/Turkey-Syria-earthquake/L1.1/BRS-HH-ALOS2449082900-220916-WBSR1.1__D-F4.jpg>

<https://jaxaalos2.s3.us-west-2.amazonaws.com/palsar2-scansar/Turkey-Syria-earthquake/L1.1/BRS-HH-ALOS2449082900-220916-WBSR1.1__D-F5.jpg>

<https://jaxaalos2.s3.us-west-2.amazonaws.com/palsar2-scansar/Turkey-Syria-earthquake/L1.1/IMG-HH-ALOS2449082900-220916-WBSR1.1__D-F1>

<https://jaxaalos2.s3.us-west-2.amazonaws.com/palsar2-scansar/Turkey-Syria-earthquake/L1.1/IMG-HH-ALOS2449082900-220916-WBSR1.1__D-F2>

<https://jaxaalos2.s3.us-west-2.amazonaws.com/palsar2-scansar/Turkey-Syria-earthquake/L1.1/IMG-HH-ALOS2449082900-220916-WBSR1.1__D-F3>

<https://jaxaalos2.s3.us-west-2.amazonaws.com/palsar2-scansar/Turkey-Syria-earthquake/L1.1/IMG-HH-ALOS2449082900-220916-WBSR1.1__D-F4>

<https://jaxaalos2.s3.us-west-2.amazonaws.com/palsar2-scansar/Turkey-Syria-earthquake/L1.1/IMG-HH-ALOS2449082900-220916-WBSR1.1__D-F5>

<https://jaxaalos2.s3.us-west-2.amazonaws.com/palsar2-scansar/Turkey-Syria-earthquake/L1.1/LED-ALOS2449082900-220916-WBSR1.1__D>

<https://jaxaalos2.s3.us-west-2.amazonaws.com/palsar2-scansar/Turkey-Syria-earthquake/L1.1/summary_ALOS2449082900-220916-WBSR1.1__D.txt>

<https://jaxaalos2.s3.us-west-2.amazonaws.com/palsar2-scansar/Turkey-Syria-earthquake/L1.1/TRL-ALOS2449082900-220916-WBSR1.1__D>

<https://jaxaalos2.s3.us-west-2.amazonaws.com/palsar2-scansar/Turkey-Syria-earthquake/L1.1/VOL-ALOS2449082900-220916-WBSR1.1__D>

Frame 2850, 2023/02/17

<https://jaxaalos2.s3.us-west-2.amazonaws.com/palsar2-scansar/Turkey-Syria-earthquake/L1.1/BRS-HH-ALOS2471852850-230217-WBSR1.1__D-F1.jpg>

<https://jaxaalos2.s3.us-west-2.amazonaws.com/palsar2-scansar/Turkey-Syria-earthquake/L1.1/BRS-HH-ALOS2471852850-230217-WBSR1.1__D-F2.jpg>

<https://jaxaalos2.s3.us-west-2.amazonaws.com/palsar2-scansar/Turkey-Syria-earthquake/L1.1/BRS-HH-ALOS2471852850-230217-WBSR1.1__D-F3.jpg>

<https://jaxaalos2.s3.us-west-2.amazonaws.com/palsar2-scansar/Turkey-Syria-earthquake/L1.1/BRS-HH-ALOS2471852850-230217-WBSR1.1__D-F4.jpg>

<https://jaxaalos2.s3.us-west-2.amazonaws.com/palsar2-scansar/Turkey-Syria-earthquake/L1.1/BRS-HH-ALOS2471852850-230217-WBSR1.1__D-F5.jpg>

<https://jaxaalos2.s3.us-west-2.amazonaws.com/palsar2-scansar/Turkey-Syria-earthquake/L1.1/BRS-HH-ALOS2471852850-230217-WBSR1.1__D-F1>

<https://jaxaalos2.s3.us-west-2.amazonaws.com/palsar2-scansar/Turkey-Syria-earthquake/L1.1/BRS-HH-ALOS2471852850-230217-WBSR1.1__D-F2>

<https://jaxaalos2.s3.us-west-2.amazonaws.com/palsar2-scansar/Turkey-Syria-earthquake/L1.1/BRS-HH-ALOS2471852850-230217-WBSR1.1__D-F3>

<https://jaxaalos2.s3.us-west-2.amazonaws.com/palsar2-scansar/Turkey-Syria-earthquake/L1.1/BRS-HH-ALOS2471852850-230217-WBSR1.1__D-F4>

<https://jaxaalos2.s3.us-west-2.amazonaws.com/palsar2-scansar/Turkey-Syria-earthquake/L1.1/BRS-HH-ALOS2471852850-230217-WBSR1.1__D-F5>

<https://jaxaalos2.s3.us-west-2.amazonaws.com/palsar2-scansar/Turkey-Syria-earthquake/L1.1/LED-ALOS2471852850-230217-WBSR1.1__D>

<https://jaxaalos2.s3.us-west-2.amazonaws.com/palsar2-scansar/Turkey-Syria-earthquake/L1.1/summary_ALOS2471852850-230217-WBSR1.1__D.txt>

<https://jaxaalos2.s3.us-west-2.amazonaws.com/palsar2-scansar/Turkey-Syria-earthquake/L1.1/TRL-ALOS2471852850-230217-WBSR1.1__D>

<https://jaxaalos2.s3.us-west-2.amazonaws.com/palsar2-scansar/Turkey-Syria-earthquake/L1.1/VOL-ALOS2471852850-230217-WBSR1.1__D>

Frame 2900, 2023/02/17

<https://jaxaalos2.s3.us-west-2.amazonaws.com/palsar2-scansar/Turkey-Syria-earthquake/L1.1/BRS-HH-ALOS2471852900-230217-WBSR1.1__D-F1.jpg>

<https://jaxaalos2.s3.us-west-2.amazonaws.com/palsar2-scansar/Turkey-Syria-earthquake/L1.1/BRS-HH-ALOS2471852900-230217-WBSR1.1__D-F2.jpg>

<https://jaxaalos2.s3.us-west-2.amazonaws.com/palsar2-scansar/Turkey-Syria-earthquake/L1.1/BRS-HH-ALOS2471852900-230217-WBSR1.1__D-F3.jpg>

<https://jaxaalos2.s3.us-west-2.amazonaws.com/palsar2-scansar/Turkey-Syria-earthquake/L1.1/BRS-HH-ALOS2471852900-230217-WBSR1.1__D-F4.jpg>

<https://jaxaalos2.s3.us-west-2.amazonaws.com/palsar2-scansar/Turkey-Syria-earthquake/L1.1/BRS-HH-ALOS2471852900-230217-WBSR1.1__D-F5.jpg>

<https://jaxaalos2.s3.us-west-2.amazonaws.com/palsar2-scansar/Turkey-Syria-earthquake/L1.1/IMG-HH-ALOS2471852900-230217-WBSR1.1__D-F1>

<https://jaxaalos2.s3.us-west-2.amazonaws.com/palsar2-scansar/Turkey-Syria-earthquake/L1.1/IMG-HH-ALOS2471852900-230217-WBSR1.1__D-F2>

<https://jaxaalos2.s3.us-west-2.amazonaws.com/palsar2-scansar/Turkey-Syria-earthquake/L1.1/IMG-HH-ALOS2471852900-230217-WBSR1.1__D-F3>

<https://jaxaalos2.s3.us-west-2.amazonaws.com/palsar2-scansar/Turkey-Syria-earthquake/L1.1/IMG-HH-ALOS2471852900-230217-WBSR1.1__D-F4>

<https://jaxaalos2.s3.us-west-2.amazonaws.com/palsar2-scansar/Turkey-Syria-earthquake/L1.1/IMG-HH-ALOS2471852900-230217-WBSR1.1__D-F5>

<https://jaxaalos2.s3.us-west-2.amazonaws.com/palsar2-scansar/Turkey-Syria-earthquake/L1.1/LED-ALOS2471852900-230217-WBSR1.1__D>

<https://jaxaalos2.s3.us-west-2.amazonaws.com/palsar2-scansar/Turkey-Syria-earthquake/L1.1/summary_ALOS2471852900-230217-WBSR1.1__D.txt>

<https://jaxaalos2.s3.us-west-2.amazonaws.com/palsar2-scansar/Turkey-Syria-earthquake/L1.1/TRL-ALOS2471852900-230217-WBSR1.1__D>

<https://jaxaalos2.s3.us-west-2.amazonaws.com/palsar2-scansar/Turkey-Syria-earthquake/L1.1/VOL-ALOS2471852900-230217-WBSR1.1__D>
