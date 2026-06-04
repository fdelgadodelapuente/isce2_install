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

Sampling frequencies of different SAR satellites. Here B$_{p}$ and PRF are the range bandwidth and the pulse repetition frequency that control the range and azimuth resolution respectively. IM6 was the default beam used to record data during the [ENVISAT extension phase](https://earth.esa.int/eogateway/missions/envisat/description). All data sets are stripmap mode except Sentinel-1 which is TOPS (Terrain Observations by Progressive Scans). The PRF can change between ALOS-1 images and even in the middle of a frame, and hence requires a modification of the coregistration methods to handle pixels of different azimuth resolutions. The look angles increases from the near to the far range slant range of the SLC. This effect is very noticeable in the Sentinel-1 TOPS data due to its wide swath acquisitions. The three numbers for TOPS data refers to swaths 1 to 3.

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

| **Satellite** | **beam/mode** | **$\Delta R_{r}$ (m)** | **$\Delta R_{a}$ (m)** | **Pixel Ratio** | **Looks** |
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

SLC images resolution by platform compared with typical available DEMs. $\Delta R_{r}$ ground range pixel size. $\Delta R_{a}$ azimuth pixel size. $^*$Many papers from ESA show that Sentinel-1 SLC data have a pixel size of 20 m in azimuth and 4-5 in range, but those are approximates numbers only! Here COP is the Copernicus DEM.

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

Note: Starting in version 2.4.0 (July 2020) all references to "reference" and "secondary" were changed to "reference" and "secondary" respectively.

```
<stripmapApp>
<component name="insar">
<property name="Sensor Name">COSMO_SKYMED</property>
<property name="demFilename">/insar_data/tandemx12m.dem</property>
<property name="reference doppler method">useDEFAULT</property>
<property name="secondary doppler method">useDEFAULT</property>
<property name="range looks">8</property> 
<property name="azimuth looks">8</property> 

<component name="reference">
<property name="HDF5">
CSKS3_RAW_B_HI_13_VV_RA_SF_20150227113114_20150227113122.h5</property>
<property name="OUTPUT">reference</property>
</component>

<component name="secondary">
<property name="HDF5">
CSKS1_RAW_B_HI_13_VV_RA_SF_20140130113244_20140130113251.h5</property>
<property name="OUTPUT">secondary</property>
</component>

<property name="filter strength">0.3</property>
<property name="do unwrap">True</property>
<property name="unwrapper name">icu</property>
<property name="regionOfInterest">[-40.64,-40.36,-72.39,-72.05]</property>
<property name="geocode bounding box">[-39.6,-39.1,-72.3,-71.6]</property>
<property name="geocode list">["interferogram/filt_topophase.flat", 
"interferogram/filt_topophase.unw","interferogram/filt_topophase.unw.conncomp", 
"interferogram/topophase.cor", "interferogram/phsig.cor", 
"geometry/los.rdr"]</property>

</component>

</stripmapApp>
```

For raw data you can skip the Doppler centroid method and the software will calculate automatically with DOPIQ (useDOPIQ), except for COSMO-SkyMed. the latter and for SLC data you have to set it to useDEFAULT.

**ERS data** requires either PRC (DPAF) or ODR (Delft) orbits. PRC orbits are better .

**ENVISAT data**

```
<property name="Sensor Name">ENVISAT</property>
<property name="reference doppler method">useDOPIQ</property>
<property name="secondary doppler method">useDOPIQ</property>

<property name="IMAGEFILE">../ASA_IM__0CNPDE20110607_035110_000000163103_00176_48466_3255.N1</property>
<property name="INSTRUMENT_DIRECTORY">/envisat/ins</property>
<property name="ORBIT_DIRECTORY">/envisat/dor_vor</property>
<property name="OUTPUT">reference</property>
```

You need either DOR (DORIS) or VOR (verified final) orbits. VOR orbits are more accurate

**ENVISAT SLC data**

```
<property name="Sensor Name">ENVISAT_SLC</property>
<property name="reference doppler method">useDEFAULT</property>
<property name="secondary doppler method">useDEFAULT</property>

<property name="IMAGEFILE">../ASA_IMS_1PNESA20080304_033012_000000182066_00304_31420_0000.N1</property>
<property name="INSTRUMENT_DIRECTORY">/envisat/ins</property>
<property name="ORBIT_DIRECTORY">/envisat/dor_vor</property>
```

Note that the ENVISAT raw filename starts with `ASA_IM__0` (LEVEL0) while the filename of ENVISAT SLC data starts with `ASA_IMS_1` (LEVEL1).

**ALOS data**

```
<property name="Sensor Name">ALOS</property>
<property name="reference doppler method">useDOPIQ</property>
<property name="secondary doppler method">useDOPIQ</property>
  
<property name="IMAGEFILE">[IMG-HH-ALPSRP273183230-H1.0__D]</property>
<property name="LEADERFILE">[LED-ALPSRP273183230-H1.0__D]</property>
<property name="RESAMPLE_FLAG">dual2single</property>
<property name="OUTPUT">reference</property>
```

The only difference for the input file is that ALOS-1 stripmap data was acquired in two different beams, FBD (fine beam double, HH-HV double polarization, 14 MHz range bandwidth) and FBS (fine beam single, HH single polarization, 28 MHz range bandwidth), resulting in twice the range resolution of the FBS beam compared with FBD (`tab:slcres`; ). Every FBD image contains one IMG-HH and one IMG-HV files, whereas a FBS image contains a single IMG-HH file. To form a usable interferogram, the images must have the same resolution, so the FBD images are zero-padded in the range direction in the frequency domain to match the length and the resolution of the FBS image. To process an interferogram with FBS and FBD images (either as reference, secondary or both), you need to include the following flag in the control file under the respective FBD image (either secondary or reference) for the FBD2FBS conversion.

FBD image to FBS

```
<property name="RESAMPLE_FLAG">dual2single</property>
```

FBS image to FBD

```
<!--<property name="RESAMPLE_FLAG">single2dual</property>-->
```

I have done test processing FBD2FBS (oversample 14 to 28 Mhz) and FBS2FBD (downsample 28 to 14 MHz). The interferograms that result are nearly equivalent and differ only by a phase constant. The split spectrum corrections are also equivalent between both products.

Note that some ALOS-1 raw images have changes in the PRF in the middle of the scene. If this happens, the image can only be processed by ISCE 2.5.0 version or newer. Alternatively you can use the old ROI_PAC with the SIO ALOS parser in GMTSAR to process them. The split spectrum method implemented in `stripmapApp.py`, only works with ALOS-1 raw data, not with ALOS-1 SLC data.

If the perpendicular baseline is longer than $\sim$0.5 km, you should process the ALOS data with a higher resolution DEM like that from Copernicus. Otherwise, if you use SRTM, the data will have several residual artifacts that result from the use of the low resolution DEM in the geometric coregistration.

You can stitch several ENVISAT and ALOS raw images just by adding more images under `IMAGEFILE` and `LEADERFILE`. The latter is an ALOS-only specific parameter.

**ALOS-2 SM1-3 data**

For ALOS-2 SM3 the data are provided as a double polarization IMG-HH and IMG-HV files with the same pixel size for each file – there is no need to run an FBD2FBS conversion. The SM1 data are provided as single polarization files with a single IMG-HH file. HH interferograms have a higher SNR than HV interferograms.

```
<property name="Sensor Name">ALOS2</property>
<property name="reference doppler method">useDEFAULT</property>
<property name="secondary doppler method">useDEFAULT</property>
       
<property name="IMAGEFILE">IMG-HH-ALOS2050286350-150429-FBDR1.1__A</property>
<property name="LEADERFILE">LED-ALOS2050286350-150429-FBDR1.1__A</property>
<property name="OUTPUT">reference</property>
```

**TerraSAR-X data**

```
<property name="Sensor Name">TERRASARX</property>
<property name="reference doppler method">useDEFAULT</property>
<property name="secondary doppler method">useDEFAULT</property>

<property name="XML">../20130717/TSX-1.SAR.L1B/
TDX1_SAR__SSC______SM_S_SRA_20130717T231121_20130717T231129/
TDX1_SAR__SSC______SM_S_SRA_20130717T231121_20130717T231129.xml</property>
<property name="OUTPUT">reference</property>
```

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

**COSMO-SkyMED raw data**

```
<property name="Sensor Name">COSMO_SKYMED</property>
<property name="reference doppler method">useDEFAULT</property>
<property name="secondary doppler method">useDEFAULT</property>

<property name="HDF5">../
CSKS3_RAW_B_HI_13_VV_RA_SF_20150227113114_20150227113122.h5</property>
<property name="OUTPUT">reference</property>
```

**COSMO-SkyMED SLC data**

```
<property name="Sensor Name">COSMO_SKYMED_SLC</property>
<property name="reference doppler method">useDEFAULT</property>
<property name="secondary doppler method">useDEFAULT</property>

<property name="HDF5">../
CSKS2_SCS_B_HI_09_HH_RA_SF_20090412050638_20090412050645.h5</property>
<property name="OUTPUT">reference</property>
```

I reccommend that you request CSK SLC data to ASI instead of raw data.

**SAOCOM-1 data**

```
<property name="Sensor Name">SAOCOM_SLC</property>
<property name="reference doppler method">useDEFAULT</property>
<property name="secondary doppler method">useDEFAULT</property>

<property name="IMAGEFILE">EOL1ASARSAO1B8776822/S1B_OPER_SAR_EOSSP__CORE_L1A_OLF_20240125T174320/Data/slc-acqId0000055332-b-sm4-2401251827-s4dp-hh</property>
<property name="XEMTFILE">EOL1ASARSAO1B8776822/S1B_OPER_SAR_EOSSP__CORE_L1A_OLF_20240125T174320.xemt</property>
<property name="XMLFILE">EOL1ASARSAO1B8776822/S1B_OPER_SAR_EOSSP__CORE_L1A_OLF_20240125T174320/Data/slc-acqId0000055332-b-sm4-2401251827-s4dp-hh.xml</property>

<property name="OUTPUT">reference</property>
```

SAOCOM-1 lacks a controlled orbital tube, so there is no guarantee that an 8- or 16-day long interferogram will have a small perpendicular baseline

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

### Ionospheric correction

Only for ALOS raw data.

```
imageMath.py -e='a_0;a_1*(c>0)-b_0*(c>0)' -s BIL  --a=interferogram/filt_topophase.unw  --b=Ionosphere/dispersive.bil.unwCor.filt  -o  interferogram/filt_topophase_nondispersive.unw  --c=interferogram/filt_topophase.conncomp
```

Here `filt_topophase.conncomp` is for an ICU unwrapped interferogram and you can replace it with `filt_topophase.unw.conncomp` for a SNAPHU_MCF unwrapped interferogram.

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

ISCE can calculate range and azimuth offsets on the coregistered SLCs. These are the same offset fields that are used to coregister the SLCs, but here they also represent the horizontal displacement field. Their accuracy is much lower compared to that of InSAR, usually 1/10 of the pixel size. Hence for SAR pixel sizes of 2-20 m, pixel offsets have accuracies of 0.2-2 m. Hence, they are only useful for very large events. Pixel offsets are particularly useful for large continental earthquakes with surface ruptures and $M_{W}$ 6.5 - 7.9. The large strain near the surface rupture results in coherence loss in most of the interferograms unless either the pixel size is very small or the radar operates with L-band. However, pixel offsets from SAR images can retrieve the deformation in those areas (e.g., ), and are not subject to either aliasing or phase unwrapping problems.

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

### Input example file

```
<topsApp> 
<component name="topsinsar"> 
<property name="Sensor Name">SENTINEL1</property>
<property name="demFilename">/dems/tandemx30m.dem</property>
<property name="geocode demFilename">/dems/tandemx30m.3alks_3rlks.dem</property>
 
<component name="reference"> 
<property name="safe">../
S1B_IW_SLC__1SDV_20191120T095727_20191120T095754_019009_023DF6_68CE.zip</property> 
<property name="output directory">"reference"</property>                 
<property name="orbit directory">/orbits/s1orb</property> 
<property name="auxiliary data directory">/orbits/s1orb</property> 
</component>
 
<component name="references"> 
<property name="safe">../
S1B_IW_SLC__1SDV_20190406T095704_20190406T095731_015684_01D6D3_D1F7.zip</property> 
<property name="output directory">"references"</property> 
<property name="orbit directory">/orbits/s1orb</property> 
<property name="auxiliary data directory">/orbits/s1orb</property> 
</component>
 
<property name="swaths">[1,2,3]</property>
<property name="ESD coherence threshold">0.85</property> <!-- above 0.7-->
<property name="do ESD">True</property>
<property name="extra ESD cycles">0</property> 
<property name="do ionosphere correction">False</property>
<property name="apply ionosphere correction">False</property>
<property name="consider burst properties in ionosphere computation">False</property>
<property name="azimuth looks">5</property>
<property name="range looks">19</property>
<property name="filter strength">0.5</property>
<property name="do unwrap">True</property>
<property name="unwrapper name">snaphu_mcf</property> <!--icu/snaphu_mcf-->
<property name="geocode bounding box">[37.48,37.97,14.76,15.02]</property>
<property name="region of interest">[-40.6,-40.5,-72.1,-72.1]</property>
<property name="geocode list">["merged/filt_topophase.unw",
"merged/filt_topophase.unw.conncomp", "merged/los.rdr",
"merged/filt_topophase.flat", "merged/topophase.cor",
"merged/phsig.cor"]</property>
 
</component>
</topsApp>
```

### Ionosphere correction

You should use this correction for TOPS data in regiones of low geomagnetic latitudes . To correct the long wavelength dispersive phase and burst overlap jumps in `topsApp.py`, add the following to the input file.

```
<property name="do ionosphere correction">True</property>
<property name="apply ionosphere correction">True</property>
<property name="consider burst properties in ionosphere computation">True</property>
```

### Extract amplitude SLCs

Must be run with the following options in the int.xml file

```
<property name="use virtual files">False</property>
<property name="geocodelist">[merged/amplitudes.bil]</property>  
```

```
#gdal_translate -of ENVI merged/reference.slc.full.vrt merged/reference.slc.full
#gdal_translate -of ENVI merged/secondary.slc.full.vrt merged/secondary.slc.full
imageMath.py -e='abs(a);abs(b)' --a=reference.slc.full --b=secondary.slc.full -t FLOAT -s BIL -o amplitudes.bil.full
looks.py -i amplitudes.bil.full -a ${alks} -r ${rlks} -o amplitudes.bil
#gdal_translate -of ENVI merged/amplitudes.bil.vrt merged/amplitudes.bil
```

### Dense offsets

One particular advantage of the TOPS mode with respect to other stripmap modes is the small pixel size in the slant range direction. Therefore if your interferogram is decorrelated due to large strain, you can still retrieve deformation from range offsets instead of interferometry (e.g., ). Due to the small pixel size in range, you can also extract more accurate range than azimuth offsets, which are particularly useful for glaciology.

[Box sizes for ampcor](https://raw.githubusercontent.com/parosen/Geo-SInC/7f89ccfa906e36c12726a663ee3d34c621797214/UNAVCO2021/4.4_Offset_stack_for_velocity_dynamics/support_files/offset_parameters.png).

```
<!-- Ssearch Window Size Height should be < 2 * Window Size Height-->
<property name="do denseOffsets">True</property>
<property name="Ampcor window width">40</property>
<property name="Ampcor window height">8</property>
<property name="Ampcor search window height">10</property>
<property name="Ampcor search window width">10</property>
<property name="Ampcor skip width">32</property>
<property name="Ampcor skip height">8</property>
```

Example for Pine Island glacier (Antarctica) from the [2021 UNAVCO ISCE Workshop.](https://github.com/parosen/Geo-SInC/blob/main/UNAVCO2021/4.4_Offset_stack_for_velocity_dynamics/nb_topsApp_offsets.ipynb)

Example for Pine Island glacier (Antarctica) from the [2023 EarthScope ISCE Workshop.](https://github.com/parosen/Geo-SInC/blob/main/EarthScope2023/4.3_Offset_stack_for_velocity_dynamics_with_autoRIFT/nb_dense_offsets.ipynb)

```
<property name="do denseoffsets">True</property>
    <property name="Filter window size">3</property>
    <property name="Ampcor window width">256</property>
    <property name="Ampcor window height">64</property>
    <property name="Ampcor search window width">40</property>
    <property name="Ampcor search window height">10</property>
    <property name="Ampcor skip width">128</property>
    <property name="Ampcor skip height">32</property>
```

Example for southern Patagonia Icefield (Chile/Argentina)

```
<property name="do denseoffsets">True</property>
    <property name="Filter window size">3</property>
    <property name="Ampcor window width">128</property>
    <property name="Ampcor window height">32</property>
    <property name="Ampcor search window width">40</property>
    <property name="Ampcor search window height">10</property>
    <property name="Ampcor skip width">16</property>
    <property name="Ampcor skip height">4</property>
```

Then multiply the range and azimuth offsets by their pixel sizes of 2.3 and 14.1 m, respectively. The offset tracking uncertainty is 1/5 to 1/10 of the pixel size, so this results in theoretical uncertainties of 0.23-0.45 m for range offsets, and 1.41-2.82 m for azimuth offsets.

<figure>
  <img src="figures/ampcor_tops.png" alt="ampcor tops" width="900">
  <figcaption><strong>Figure.</strong> Range and azimuth offsets for 12 day Sentinel-1 pair.</figcaption>
</figure>


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

The ALOS-2 processor was released in October 2019 as an additional toolbox to ISCE 2.3.2 and was properly integrated with the rest of the ISCE modules in version 2.3.3 in March 2020. It can process both stripmap and ScanSAR data with split spectrum corrections . ScanSAR images suitable for InSAR are focused with a full aperture processing chain, and not with a SPECAN algorithm whoch is ideal for these burst by burst acquisition modes. Although `stripmapApp.py` can process ALOS-2 SM3 and SM1 data, it cannot correct the ionospheric phase. The module tutorial and examples are at

```
isce2-2.6.3/examples/input_files/alos2/example_input_files
```

To process an interferogram

```
alos2App.py alos2app.xml --steps
```

The ScanSAR processing is time consuming and requires $\sim$60 Gb of storage for every frame. Every ALOS-2 ScanSAR swath is $\sim$6 Gb and every interferograms must be calculated three times for the range spectral filtering to separate the dispersive and non-dispersive components of the phase. In my experience ScanSAR data for volcanic applications is only useful for large scale surveys of volcanic deformation. For example, a single ScanSAR swath in two frames can cover a almost all of the most active volcanoes of the Southern Andes. And obviously when there is no other coherent data that span the event of interest. The advantages of ScanSAR for large earthquakes are obvious like during the 2015 Gorkha  and the 2016 Kaikoura  earthquakes.

For processing ScanSAR data the tutorial recommends using 5 looks in range and 2 looks in azimuth. The pixel sizes are 25 m in range – similar to the SM3 pixel size, and 60 m in azimuth. This results in a pixel size of $\sim$100 m, so the interferograms are geocoded with the 90 m DEM. You can stitch ScanSAR frames and process as many swaths and frames as you want.

For the SM3 data the pixel ratio is 2, but alos2App.py applies by default the pixel ratio and 2 additional looks in range and azimuth. These data are usually processed with either 2 or 4 additional looks resulting in a total of 16 looks in azimuth and 8 looks in range.

### Technical notes from JAXA

The following links detail known issues with ALOS-2 data.

[Effective data for interferometric analysis with PALSAR-2 ScanSAR mode](https://www.eorc.jaxa.jp/ALOS/en/alos-2/pdf/auig2/ScanSAR_Burst_Overlap_20151127_e.pdf). ScanSAR interferometry is not possible with data acquired before February 2015. More details in . But ScanSAR to stripmap interferometry before February 2015 is possible (e.g., ).

[Change of the center frequency for the beam F2-6 in Stripmap Fine \[10 m\] mode](https://www.eorc.jaxa.jp/ALOS/en/alos-2/pdf/auig2/AUIG2_CenterFrequency_20151127_e.pdf). If your data is from the F2-6 stripmap beam, you cannot calculate stripmap interferograms with images acquired before and after June 01 2015. Several stripmap tracks have this issue. More details in .

[Correction of the range offset error in Stripmap \[10 m\] and ScanSAR \[350 km / 490 km\] modes](https://www.eorc.jaxa.jp/ALOS/en/alos-2/pdf/auig2/Update_ALOS2_RangeOffset_20181122_En.pdf). Split spectrum corrections are not possible with data acquired before November 2018.

### ALOS-2 files naming convention

[ALOS-2/PALSAR-2 Level 1.1/1.5/2.1/3.1 CEOS SAR Product Format Description](https://www.eorc.jaxa.jp/ALOS-2/en/doc/fdata/PALSAR-2_xx_Format_CEOS_E.pdf)

ALOS-2 has 15 observation modes (SBS, UBS, UBD, HBS, HBD, HBQ, FBS, **FBD**, FBQ, **WBS**, **WBD**, WWS, WWD, VBS, VBD) and the relevant modes are stripmap FBD fine mode (dual polarization), WBS and WBD ScanSAR nominal \[14MHz\] mode (single and dual polarization).

    IMG-HH-ALOS2471852900-230217-WBSR1.1__D-F3

`HH` polarization (reception - emission, here both Horizontal, can also be HV Horizontal and Vertical for double polarization mode)

`ALOS2` sensor

`47185` Orbit accumulation number of a scene center

`2900` frame

`230217` acquisition date, YYMMDD

`WBS` acquisition mode. WBS is ScanSAR nominal \[14MHz\] mode Single polarization. 14 MHz is the range bandwith that results in a ground range pixel size of $\sim$20 m/pixel. It can also be `WBD` which is ScanSAR nominal \[14MHz\] mode Double polarization. This results in a HH and a HV file for every swath.

`R` observation direction, right looking

`1.1` processing level, range-azimuth single look complex.

`D` descending

`F3` swath 3

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

### SM3 and SM1 processing in `alos2App.py` compared with `stripmapApp.py`

Both `alos2App.py` and `stripmapApp.py` can process SM3 and SM1 data. The phase difference of a multilooked unfiltered interferogram processed with `alos2App.py` and the same interferorgam processed with `stripmapApp.py` is approximately a constant value smaller than $\pi$. This phase difference is accounted for when you remove a ramp as part of source modeling. Hence, both workflows produce equivalent interferograms for geological and geophysical applications. Of course the phase difference will be a ramp if you apply the split spectrum correction available in `alos2App.py`.

### Ionospheric correction for SM3 data

Ionospheric corrections only work with SM3 SLCs with the [range offset error](https://www.eorc.jaxa.jp/ALOS-2/en/calval/Update_ALOS2_RangeOffset_20181122_En.pdf) fixed. These are data provided by JAXA after November 2018. If your SLC was processed before this date, then the software cannot correct the dispersive phase.

If the split spectrum correction for SM3 data fails, you can improve it with the following:

```
<property name="number of range looks ion">16</property>
<property name="number of azimuth looks ion">16</property>

<property name="maximum window size for filtering ionosphere phase">151</property>
<property name="minimum window size for filtering ionosphere phase">51</property>
```

Increasing the number of looks, and window size for filtering the ionosphere phase, avoided anomalous short wavelength ionospheric fringes in the correction. However, if the window size is too large, then the polynomial contribution of the ionosphere correction will dominate.

If the `ion/ion_cal/ion_80rlks_448alks.ion` file displays a jump between the swaths, then this isue will propagate into the final corrected interferogram. To fix it, open the `alos2App.py` input file, and add the following

```
<property name="swath phase difference snap to fixed values">[[True, True, True, False]]</property>
```

Here the False flag refers to the fourth and fifth swaths, which is the one that shows the phase discontinuity. Then, restart the processing from `ion_subband`.

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

### Google Earth KMZ

To create a Google Earth kml file

```
mdx.py filt_topophase.unw.geo -kml filt_topophase.unw.geo.kml
```

If you move the output kml and png files to another folder, you will have to manually update the file path with a text editor. To remove the amplitude do the following:

```
mdx filt_topophase.unw.geo -s 5798 -unw -r4 -rhdr 23192 -cmap cmy -wrap 6.28318 -P
```

Here `s` is the file width and `rhdr` is the size of the file header at the start of each line times 4. This outputs a ppm file with the phase only. Then, remove the cyan background with

```
convert out.ppm -transparent cyan filt_topophase.unw.geo.png
```

Finally edit the .kml file in a text editor to remove the absolute path to the image file.

### Heading and look angle file

Historically, people did not all use the same sign conventions in InSAR. Check whether your interferograms are ‘range change’ or ‘ground LOS displacement’. Check if your pointing vectors are consistent with your interferograms (pointing from satellite to ground, or ground to satellite?).

ISCE does not output heading (azimuth) direction in a geographical convention. It uses a right-hand rule from east (i.e. 0=east, and counts positive degrees counter-clockwise from there). To convert to geographical heading, multiply the azimuth by -1 and add 90$º$.

[Math formula to generate the directional cosines from the LOS file directly with the software](http://earthdef.caltech.edu/boards/4/topics/327). This is equivalent to formulas implemented in the `load\_isce.m` scripts.

```
imageMath.py --eval='sin(rad(a_0))*cos(rad(a_1+90));  sin(rad(a_0)) * sin(rad(a_1+90)); cos(rad(a_0))'  --a=los.rdr.geo -t FLOAT -s BIL -o enu.rdr.geo
```

$$S_{1} = cos(h+90)sin(l); S_{2} = sin(h+90)sin(l); S_{3} = cos(l)$$ Here $l$ is the look angle and $h$ is the satellite heading, $S_i$ are the directional cosines and the line-of-sight displacement $U_{LOS}$ is

$$U_{LOS} = \vec{S} \bullet \vec{U} = S_{1}U_{x} + S_{2}U_{y} + S_{3}U_{z}$$ with $U_{i}$ the displacement in the $_{i}$ direction. For pressure sources like those in volcanoes most the displacement projected into the LOS is vertical so it is standard practice to describe those signals as either "uplift" or "subsidence", even though it is incorrect from the line-of-sight point of view. For earthquakes both the east-west and vertical components can have equal magnitudes, so in the case it is more accurate to talk about line-of-sight displacement increase/decrease.

## DEM generation from TanDEM-X CoSSC data

The CoSSC processing is only partially implemented in the software. ISCE can process a bistatic interferogram up to the unwrapping step with scripts available only at JPL, but it does not have a module to accurately reconstruct the topography from the phase in slant range. Also, the bistatic geometry does not take into account in the range-Doppler equations for the geocoding. Neglecting these corrections is not very important if your topographic change is less than $\sim$50 m, but can produce very obvious errors in the elevation and the geocoding if you have large topographic changes ($>$ 150 m). You can find examples of bistatic processing with ISCE in . The only software that I am aware of that can process TanDEM-X CoSSC data are the DLR GENESIS processor, [GAMMA](https://www.gamma-rs.ch), and a modified version of DORIS by the Karlshruhe Institute of Technology.

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

# Stack processors

## Stripmap Stack Processor

### ALOS raw data

Download the ALOS data from ASF to a folder called download, and unpack it with the ISCE parser.

```
prepRawALOS.py -i download/ -o SLC
prepRawSensor.py -i download/ -o SLC
```

If you do not want to apply the ionospheric correction, then zero pad the 14 MHz FBD images to match the 28 MHz FBS data. Otherwise, the parser will downsample the FBS data to FBD range sampling, so the split spectrum correction is applied to the common range bandwidth.

```
prepRawALOS.py -i download/ -o SLC --dual2single --fbd2fbs
```

Run the commands in `run_unPackALOS file`.

```
source ~/.bash_profile; unpackFrame_ALOS_raw.py -i  /home/fdelgado/insarproc/hudson/download/20070304  -o /home/fdelgado/insarproc/hudson/SLC/20070304 -f  fbs2fbd  -m

source ~/.bash_profile; unpackFrame_ALOS_raw.py -i  /home/fdelgado/insarproc/hudson/download/20071020  -o /home/fdelgado/insarproc/hudson/SLC/20071020 -m

source ~/.bash_profile; unpackFrame_ALOS_raw.py -i  /home/fdelgado/insarproc/hudson/download/20071205  -o /home/fdelgado/insarproc/hudson/SLC/20071205 -f  fbs2fbd  -m

source ~/.bash_profile; unpackFrame_ALOS_raw.py -i  /home/fdelgado/insarproc/hudson/download/20090309  -o /home/fdelgado/insarproc/hudson/SLC/20090309 -f  fbs2fbd  -m

source ~/.bash_profile; unpackFrame_ALOS_raw.py -i  /home/fdelgado/insarproc/hudson/download/20100125  -o /home/fdelgado/insarproc/hudson/SLC/20100125 -f  fbs2fbd  -m

source ~/.bash_profile; unpackFrame_ALOS_raw.py -i  /home/fdelgado/insarproc/hudson/download/20100612  -o /home/fdelgado/insarproc/hudson/SLC/20100612 -m

source ~/.bash_profile; unpackFrame_ALOS_raw.py -i  /home/fdelgado/insarproc/hudson/download/20100728  -o /home/fdelgado/insarproc/hudson/SLC/20100728 -m

source ~/.bash_profile; unpackFrame_ALOS_raw.py -i  /home/fdelgado/insarproc/hudson/download/20110128  -o /home/fdelgado/insarproc/hudson/SLC/20110128 -f  fbs2fbd  -m

source ~/.bash_profile; unpackFrame_ALOS_raw.py -i  /home/fdelgado/insarproc/hudson/download/20110315  -o /home/fdelgado/insarproc/hudson/SLC/20110315 -f  fbs2fbd  -m
```

To properly apply the split spectrum correction you need to process the data in the common spectral band, which means that all the FBS (28 MHz) images must be downsampled to the FBD (14 MHz) mode , so the pixel ratio is 4. If you process the data with `-f fbd2fbs` then the ionospheric correction might not work properly. In this example I process the data with 8 looks in range for the `FBD` pixel size, resulting in a pixel size of $\sim$ 160 m.

Run `stackStripmap.py`.

```
stackStripMap.py -W ionosphere -s SLC  -d /home/fdelgado/insarproc/dem/srtm_svz30m.dem  -m 20071205 -t 1500  -b 2000 -a 32 -r 8 -u snaphu -f 0.5
```

Here `-W` is the workflow, `-s` is the folder with the raw images extracted by the ISCE parser, `-d` is the DEM file name, `-m` is the reference image with respect to which all the other images will be aligned to, `-t` is the temporal baseline, `-a` is the azimuth looks, `-r` are the range looks, `-u` is the unwrapping module – either icu or snaphu, and `-f` is the power spectrum filtering strength.

`stackStripmap.py` will generate many configuration files that need to be manually executed. It also generates a perpendicular baseline plot `pairs.pdf` (`fig:bp`).

<figure id="fig:bp">
  <img src="figures/pairs.png" alt="pairs" width="900">
  <figcaption><strong>Figure.</strong> Perpendicular baseline plot for ALOS data. The reference image is not the earliest image.</figcaption>
</figure>


<figure id="fig:bp_alos114">
  <img src="figures/pairs_alos_p114.png" alt="pairs alos p114" width="900">
  <figcaption><strong>Figure.</strong> Perpendicular baseline plot for ALOS path 114. The reference image is not the earliest image. Note the systematic drift in the satellite orbit from 2007 to mid 2008 and then from late 2008 until the end of the mission in April 2011.</figcaption>
</figure>


The specific DEM files depend on the data set. Usually it is SRTM but if you have a better DEM like TanDEM-X 12 m you need to change it with the `-d` flag. Now run the following files

```
sh run_files/run_1_reference
sh run_files/run_2_focus_split
sh run_files/run_3_geo2rdr_coarseResamp
sh run_files/run_4_refinesecondaryTiming
sh run_files/run_5_invertMisreg
sh run_files/run_6_fineResamp
sh run_files/run_7_denseOffset
sh run_files/run_8_invertDenseOffsets
sh run_files/run_9_resampleOffset
sh run_files/run_10_replaceOffsets
sh run_files/run_11_fineResamp
sh run_files/run_12_grid_baseline
sh run_files/run_13_igram
sh run_files/run_14_igramLowBand
sh run_files/run_15_igramHighBand
sh run_files/run_16_iono
```

You can run them all with

```
ls  | awk '{print "sh",$1}' | csh
```

Run the stack processor, do not apply the dense offsets to improve the geometric coregistration, and apply the split spectrum correction.

```
sh run_files/run_1_reference
sh run_files/run_2_focus_split
sh run_files/run_3_geo2rdr_coarseResamp
sh run_files/run_4_refinesecondaryTiming
sh run_files/run_5_invertMisreg
sh run_files/run_6_fineResamp
sh run_files/run_7_denseOffset
sh run_files/run_8_invertDenseOffsets
sh run_files/run_9_resampleOffset
sh run_files/run_11_fineResamp
sh run_files/run_12_grid_baseline
sh run_files/run_13_igram
sh run_files/run_14_igramLowBand
sh run_files/run_15_igramHighBand
sh run_files/run_16_iono
```

Run the stack processor with split spectrum correction, and no dense offsets calculation.

```
sh run_files/run_1_reference
sh run_files/run_2_focus_split
sh run_files/run_3_geo2rdr_coarseResamp
sh run_files/run_4_refinesecondaryTiming
sh run_files/run_5_invertMisreg
sh run_files/run_6_fineResamp
sh run_files/run_12_grid_baseline
sh run_files/run_13_igram
sh run_files/run_14_igramLowBand
sh run_files/run_15_igramHighBand
sh run_files/run_16_iono
```

If you want to calculate the perpendicular baseline for all the processed interferograms, you can store them as text files in the `baselines` directoy

```
ls Igrams/202* -d1 | awk '{print "baseline.py -m SSC/SLCS/"substr($0,8,8),"-s SSC/SLCS/"substr($0,17,8)" | grep Baseline > baselines/ifg_"substr($0,8,8)"_"substr($0,17,8)}' > t.csh; csh t.csh
```

After several hours or days depending upon the computer, the software will output a set of unwrapped interferograms. The ionospheric correction is not applied automatically, so you can apply it with `imageMath.py`.

```
imageMath.py -e='a_0;a_1-b_0' -s BIL --a=Igrams/20071205_20100612/filt_20071205_20100612_snaphu.unw --b=Ionosphere/20071205_20100612/iono.bil.unwCor.filt  -o Igrams/20071205_20100612/filt_20071205_20100612_snaphu_nondispersive.unw 
```

You can also use awk to apply this command to all the interferograms

For ICU-unwrapped interferograms

```
ls Igrams/2*/*icu.unw | awk '{print "imageMath.py  -e=\47a_0;a_1*(c>0)-b_0*(c>0)\47 -s BIL  --a="$1,"--b=Ionosphere/"substr($0,8,18)"iono.bil.unwCor.filt","-o  "substr($0,1,51)"_nondispersive.unw --c="substr($0,1,47)".conncomp"}' 
```

For SNAPHU_MCF-unwrapped interferograms

```
ls Igrams/2*/*snaphu.unw | awk '{print "imageMath.py  -e=\47a_0;a_1*(c>0)-b_0*(c>0)\47 -s BIL  --a="$1,"--b=Ionosphere/"substr($0,8,18)"iono.bil.unwCor.filt", "-o  "substr($0,1,54)"_nondispersive.unw  --c="substr($0,1,47)"_snaphu.unw.conncomp"}'
```

<table>
<tr>
<td><img src="figures/dispersive.png" width="300"></td>
<td><img src="figures/ion.png" width="300"></td>
<td><img src="figures/nondispersive.png" width="300"></td>
</tr>
</table>

**Figure.** [a] ALOS interferogram of the Mw 7.0 Pichilemu earthquake with ionospheric streaks (`Igrams/20100309_20100424/filt_20100309_20100424_snaphu.unw`). [b] Ionospheric dispersive phase predicted by the split-spectrum correction that uses sub-band interferograms (`Ionosphere/20100309_20100424/iono.bil.unwCor.filt`). [c] Corrected interferogram after removal of the dispersive ionospheric phase (`Igrams/20100309_20100424/filt_20100309_20100424_snaphu_nondispersive.unw`).



Take looks on the line-of-sight file

```
looks.py -i merged/geom_reference/los.rdr -a 32 -r 8
```

The resulting file `los.32alks.8rlks.rdr` is the same one than `geometry/los.rdr` generated by `stripmapApp.py` when processing an interferogram with the same reference image and number of looks. You can also use the file `geom_reference/los.rdr` file. This file is in the ENVI file format instead of ISCE’s native RMG format, so you will have to convert it with `gdal_translate`.

```
gdal_translate los.rdr -Of ISCE los.r4
```

Geocode the relevant files

```
geocode.py -a 32 -r 8 -d /Users/francisco/insarproc/dem/srtm_svz90m.dem 
-m SLC/20071205 -i
Igram/20161008_20161217/filt_20161008_20161217_snaphu.unw 
-b -46.44 -45.63 -73.69 -72.48

geocode.py -a 32 -r 8 -d /Users/francisco/insarproc/dem/srtm_svz90m.dem 
-m SLC/20071205 -i 
Igram/20161008_20161217/20161008_20161217.cor 
-b -46.44 -45.63 -73.69 -72.48

geocode.py -a 32 -r 8 -d /Users/francisco/insarproc/dem/srtm_svz90m.dem 
-m SLC/20071205 -i 
merged/geom_reference/los.32alks_8rlks.rdr 
-b -46.44 -45.63 -73.69 -72.48
```

Here `-b` is the geocoding bounding box and the rest of the parameters as in `stackStripmap.py`

You can also use `geocodeGdal.py`. Here the coordinates are for Kilauea, so you only have to change the bounding box for your area of interest. You can geocode to whatever posting you want. For example here I geocoded to a posting of 90 m/pixel or 3 arc seconds (1/1200 = 0.0008333333333333334).

```
geocodeGdal.py -l ../merged/geom_reference/lat.rdr -L
../merged/geom_reference/lon.rdr -f 
20070620_20071221/filt_20070620_20071221_icu.unw  -b
'19.2 19.6 -155.43 -154.93' -x 0.0008333333333333334 -y 0.0008333333333333334
```

A small set of the 6 ALOS images of the 2010 Eyjafjallajökull eruption without ionospheric corecction. Here I convert all the FBS to FBD images so they have a common range band for the split spectrum correction. I also use the open source [90 m TanDEM-X DEM](https://download.geoservice.dlr.de/TDM90/) because the default SRTM does not cover latitudes higher than 60 degrees, like those of Iceland. I stitched the DEM with GDAL and then I copied the relevant file dimensions to the .xml and .vrt files with the metadata.

```
unpackFrame_ALOS_raw_2.py  -i download/20071006 -o SLC/20071006
unpackFrame_ALOS_raw_2.py  -i download/20071121 -o SLC/20071121 -f fbs2fbd
unpackFrame_ALOS_raw_2.py  -i download/20080106 -o SLC/20080106 -f fbs2fbd
unpackFrame_ALOS_raw_2.py  -i download/20100413 -o SLC/20100413 -f fbs2fbd
unpackFrame_ALOS_raw_2.py  -i download/20100529 -o SLC/20100529 -f fbs2fbd
unpackFrame_ALOS_raw_2.py  -i download/20100714 -o SLC/20100714 -f fbs2fbd

stackStripMap.py -s SLC/ -d tdx90m.dem -t 1000 -b 1000 -a 16 -r 4 -u snaphu 
-W ionosphere -f 0.5 -m 20100413

geocode.py -a 16 -r 4 -d tdx90m.dem 
-m SLC/20100413 -i 
Igrams/20100413_20100714/filt_20100413_20100714_snaphu.unw 
-b 63.35 64.38 -20.16 -18.25
```

If you want to print the the metadata of a stripmap SLC with Python3

```
import shelve
s = shelve.open('data')
import isce
import isceobj
s2 = s['frame']
dir(s2)
```

### ERS, ENVISAT, COSMO-SkyMED raw data

The workflow is almost the same than for ALOS data. First, you need to dump the ENVISAT `ASA_IM_*.N1` and COSMO-SkyMED `CSKS*.h5` files into folders that have the image date as folder name. For TerraSAR-X you need to unzip each `TSX*SSC.tar.gz` file to extract the resulting `dims_op_oc_dfd2` folder. Then, unpack every image with

```
unpackFrame_ERS_raw.py  -i download/20070504 -o SLC/20070504 [-m]
unpackFrame_ENV_raw.py  -i download/20070504 -o SLC/20070504 [-m]
unpackFrame_CSK_raw.py  -i download/20070504 -o SLC/20070504
```

The `-m` flag is optional and allows you to merge different ENVISAT frames (if you need to stitch them and only for raw data). Now run `stackStripMap.py`. Here I use an example of ENVISAT IM6 data that have a pixel ratio of 3.

```
stackStripMap.py  -W interferogram  -s SLC 
-d /data/francisco/srtm_svz30m.dem -t 1500 -b 400 -a 20 -r 4  -u snaphu/icu 
-f 0.5 -m slcs/20050803
```

Then run the rest of the workflow as for ALOS raw data. As the X and C-band data are not really sensitive to the ionosphere, there is no need to run the split spectrum and rubber sheeting steps.

```
sh run_files/run_1_reference
sh run_files/run_2_focus_split
sh run_files/run_3_geo2rdr_coarseResamp
sh run_files/run_4_refinesecondaryTiming
sh run_files/run_5_invertMisreg
sh run_files/run_6_fineResamp
sh run_files/run_7_grid_baseline
sh run_files/run_8_igram
```




<figure id="fig:bp_envi_selected">
  <img src="figures/is2_a320.png" alt="is2 a320" width="900">
  <figcaption><strong>Figure.</strong> Perpendicular baseline plot for ENVISAT IM2 ascending track 320 at Yellowstone caldera ((Delgado2021a)) with selected interferograms and SLCs classified as either winter (blue circles) or non winter (red circles based on whether they result in low coherence interferograms or not. The figure does not include winter radar images. Note the satellite orbit spread from 2004 until early 2007. Afterwards the orbits were much better controlled.</figcaption>
</figure>


### ERS, ENVISAT, TerraSAR-X, COSMO-SkyMED, RADARSAT-2 and ALOS-2 stripmap SLC data

Unpack every image with `unpackFrame_SENSOR.py`. Here SENSOR is the name of the satellite. Unlike raw data, you cannot merge different stripmap frames if the data is provided as SLC because the different frames are focused with a specific set of parameters, including a Doppler centroid that is optimized for each image. For example, focusing parameters of a SLC might not be consistent with that of the following frame.

```
unpackFrame_ERS.py  -i download/20070504 -o SLC/20070504
unpackFrame_ENV.py  -i download/20070504 -o SLC/20070504
unpackFrame_CSK.py  -i download/20070504 -o SLC/20070504
unpackFrame_TSX.py  -i dims_op_oc_dfd2_661434447_1 -o SLC/20210221
unpackFrame_PAZ.py -i dims_op_oc_dfd2_661434447_1 -o SLC/20210221
unpackFrame_RSAT2.py -i tiff/20180119  -o SLC/20180119
unpackFrame_ALOS2.py -i download/20180119  -o SLC/20180119
unpackFrame_SAOCOM.py -i EOL1A/20220201_EOL1ASARSAO1B10547757 -o ~/saocom/colina/SLC/20220201
```

Run `stackStripmap.py`. The only difference is that here you will not focus the data, so you must set the software to read the data as zero-Doppler SLCs with the variable `-z`.

```
stackStripMap.py -W interferogram -z --nofocus -s slcs -m 20180315 -d 
/Users/francisco/Documents/dems/TanDEM-X/isce_dems/tandemx30m.dem 
-t 366 -b 200 -a 15 -r 15 -u snaphu 
```

The specific number of range and azimuth looks depends upon the satellite. The rest of the workflow is the same as in the previous examples.

<figure id="fig:bp_rs2">
  <img src="figures/pairs_rs2_u16w2.png" alt="pairs rs2 u16w2" width="900">
  <figcaption><strong>Figure.</strong> Perpendicular baseline plot for RADARSAT-2 data from beam U16W2 ((Delgado2021)).</figcaption>
</figure>


<figure id="fig:bp_csk">
  <img src="figures/pairs_csk_villarrica.png" alt="pairs csk villarrica" width="900">
  <figcaption><strong>Figure.</strong> Perpendicular baseline plot for COSMO-SkyMed descending data for Villarrica volcano ((Delgado2017)). Note the large spread in baselines compared with either ENVISAT (fig:bp_envi) or RADARSAT-2 (fig:bp_rs2). A comparison with an ALOS track is not direct because the ALOS missions had a systematic orbit drift and reset (fig:bp_alos114) that was not enforced for other satellites.</figcaption>
</figure>


<figure id="fig:bp_csk">
  <img src="figures/alos2_saocom_Bp.png" alt="alos2 saocom Bp" width="900">
  <figcaption><strong>Figure.</strong> Perpendicular baseline plot for ALOS-2 SM3 and SAOCOM-1 stripmap data ((Delgado2024)). SAOCOM-1 lacks a controlled orbital tube, so that results in large baselines compared with ALOS-2.</figcaption>
</figure>


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


### Example for the 2020 Nima M$_{W}$ 6.4 earthquake, path 121 descending 

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

## ALOS-2 Stack Processor

The ALOS-2 stack processor was released in ISCE 2.5.0, March 2021. It includes the split spectrum correction for SM3, WD1 and SM3-WD1 data. The main difference with the stripmap stack processor is that the ALOS-2 stack processor includes the split spectrum ionospheric correction, while the former does not.

The tutorials are located at

```
isce/isce2-2.5.3/contrib/stack/alosStack/alosStack_tutorial.txt
isce/isce2-2.5.3/contrib/stack/alosStack/alosStack.xml
```

You need to copy and modify the `alosStack.xml` file. The file structure of `alosStack.xml` is very similar to that of the input file of `alos2App.py`. You can run them with

```
create_cmds.py -stack_par alosStack.xml
./cmd_1.sh
./cmd_2.sh
./cmd_3.sh
```

You need to manually check the quality of the ionospheric correction before running the two last steps of

```
./cmd_3.sh
```

If the correction was successful, you need to uncomment these two steps and run them in a separate script.

```
/home/fdelgado/isce/isce2-2.5.3/contrib/stack/alosStack/ion_ls.py 
-idir pairs_ion -odir dates_ion -ref_date_stack 171201 
-nrlks1 2 -nalks1 4 -nrlks2 4 -nalks2 4 -nrlks_ion 16 
-nalks_ion 16 -interp


insarpair=($(ls -l pairs | grep ^d | awk '{print $9}'))
for ((i=0;i<${#insarpair[@]};i++)); do

  IFS='-' read -ra dates <<< "${insarpair[i]}"
  ref_date=${dates[0]}
  sec_date=${dates[1]}

  cd pairs
  cd ${insarpair[i]}
  #uncomment to run this command
  /home/fdelgado/isce/isce2-2.5.3/contrib/stack/alosStack/ion_correct.py 
  -ion_dir ../../dates_ion -ref_date ${ref_date} 
  -sec_date ${sec_date} -nrlks1 2 -nalks1 4 -nrlks2 4 
  -nalks2 4
  cd ../../

done

```

The specific parameters and file paths depend upon each data set.

Afterwards filter, unwrap and geocode as in the rest of the workflows.

```
./cmd_4.sh
```

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

# Time Series Analysis

## MintPy

The Miami insar time series Python code is a software that will run time series. Unlike most of the time series software, it applies corrections (ramps, DEM errors, atmospheric phase delays) in the time series domain after the inversion . In constrast, other workflows like GIAnT apply these corrections in the interferogram domain before the displacement inversion.

If you use ALOS, you need to apply the ionospheric correction before importing the data into MintPy.

For data sets processed with `stackStripMap.py`, pick an SLC and extract the metadata into a ROI_PAC .rsc file.

    prep_isce.py -d ./Igrams -m merged/SLC/20070131/referenceShelve/data.dat 
    -b ./baselines -g ./geom_reference

Create a new folder, dump the `smallbaselineApp.cfg` file here, edit it for the relevant directories, and run the inversion.

    smallbaselineApp.py smallbaselineApp.cfg

# Example interferograms of volcanoes and earthquakes from ENVISAT, ALOS, Sentinel-1, and ALOS-2

## Volcanoes

### Yellowstone caldera uplift ENVISAT

<figure>
  <img src="figures/yellowstone.png" alt="yellowstone" width="900">
  <figcaption><strong>Figure.</strong> Yellowstone, ENVISAT 2006/08/23 - 2008/10/01.</figcaption>
</figure>


<https://imaging.unavco.org/data/sar/lts/export/ENV1/320/891/ASA_IM__0CNPDE20060823_050258_000000182050_00320_23420_2023.N1>

<https://imaging.unavco.org/data/sar/lts/winsar/ENV2/320/873/ASA_IM__0CNPDE20081001_050242_000000212072_00320_34442_6892.N1>

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

### Kilauea, June 2007 Father’s Day dike intrusion, ENVISAT/ALOS

ENVISAT C-band track 136 ascending IM4

<https://eo-virtual-archive4.esa.int/supersites/ASA_IM__0CNPDK20070412_082532_000000162057_00136_26743_2801.N1>

<https://eo-virtual-archive4.esa.int/supersites/ASA_IM__0CNPDK20070621_082536_000000172059_00136_27745_3245.N1>

ENVISAT C-band track 93 ascending IM2

<https://eo-virtual-archive4.esa.int/supersites/ASA_IM__0CNPDK20070514_081953_000000172058_00093_27201_3022.N1>

<https://eo-virtual-archive4.esa.int/supersites/ASA_IM__0CNPDK20070618_081954_000000162059_00093_27702_3230.N1>

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


### Calbuco, April 22 2015 eruption, Sentinel-1

Sentinel-1A C-band ascending interferogram

<https://datapool.asf.alaska.edu/SLC/SA/S1A_IW_SLC__1SSV_20150414T234157_20150414T234224_005486_007013_5C44.zip>

<https://datapool.asf.alaska.edu/SLC/SA/S1A_IW_SLC__1SDV_20150426T234151_20150426T234227_005661_007430_AFBE.zip>

<figure>
  <img src="figures/s1_calbuco.png" alt="s1 calbuco" width="900">
  <figcaption><strong>Figure.</strong> Calbuco eruption, Sentinel-1 ascending 2015/04/14 - 2015/04/26.</figcaption>
</figure>


Sentinel-1A C-band descending interferogram

<https://datapool.asf.alaska.edu/SLC/SA/S1A_IW_SLC__1SSV_20150421T095746_20150421T095813_005580_00723B_814B.zip>

<https://datapool.asf.alaska.edu/SLC/SA/S1A_IW_SLC__1SDV_20150503T095746_20150503T095822_005755_007640_67BE.zip>

### Nyiragongo, May 2021 eruption, Sentinel-1

Sentinel-1 C-band ascending interferogram

<https://datapool.asf.alaska.edu/SLC/SB/S1B_IW_SLC__1SDV_20210519T162006_20210519T162034_026975_03390E_40B8.zip>

<https://datapool.asf.alaska.edu/SLC/SB/S1B_IW_SLC__1SDV_20210519T162032_20210519T162103_026975_03390E_214E.zip>

<https://datapool.asf.alaska.edu/SLC/SA/S1A_IW_SLC__1SDV_20210525T162044_20210525T162111_038046_047D88_B2CD.zip>

<https://datapool.asf.alaska.edu/SLC/SA/S1A_IW_SLC__1SDV_20210525T162109_20210525T162135_038046_047D88_17E6.zip>

Sentinel-1 C-band descending interferogram

<https://datapool.asf.alaska.edu/SLC/SB/S1B_IW_SLC__1SDV_20210521T034430_20210521T034458_026997_0339AB_E9E4.zip>

<https://datapool.asf.alaska.edu/SLC/SA/S1A_IW_SLC__1SDV_20210527T034526_20210527T034555_038068_047E2D_A805.zip>

## Earthquakes

### March 2008 M$_{W}$ 7.2 Yutian, normal faulting, ALOS

ALOS-1 L-band ascending interferogram (2 frames)

<https://datapool.asf.alaska.edu/L1.0/A3/ALPSRP111100690-L1.0.zip>

<https://datapool.asf.alaska.edu/L1.0/A3/ALPSRP111100700-L1.0.zip>

<https://datapool.asf.alaska.edu/L1.0/A3/ALPSRP124520690-L1.0.zip>

<https://datapool.asf.alaska.edu/L1.0/A3/ALPSRP124520700-L1.0.zip>

<figure>
  <img src="figures/yutian_20080224_20080526_2frames.png" alt="yutian 20080224 20080526 2frames" width="900">
  <figcaption><strong>Figure.</strong> 2008 Yutian earthquake, normal faulting, ALOS ascending 2008/02/24 - 2008/05/26.</figcaption>
</figure>


### March 2010 M$_{W}$ 6.9 Pichilemu, normal faulting, ALOS

ALOS-1 L-band ascending interferogram (2 frames, `fig:alos_pichilemu`)

<https://datapool.asf.alaska.edu/L1.0/A3/ALPSRP219546490-L1.0.zip>

<https://datapool.asf.alaska.edu/L1.0/A3/ALPSRP219546480-L1.0.zip>

<https://datapool.asf.alaska.edu/L1.0/A3/ALPSRP226256490-L1.0.zip>

<https://datapool.asf.alaska.edu/L1.0/A3/ALPSRP226256480-L1.0.zip>

### June 2015 M$_{W}$ 6.3 Pishan, thrust faulting, Sentinel-1

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

### April 2015 M$_{W}$ 7.8 Gorkha, thrust faulting

[ALOS-2 interferograms](https://topex.ucsd.edu/nepal/)

### November 2016 M$_{W}$ 7.8 Kaikoura, strike-slip faulting

[ALOS-2 interferograms](https://topex.ucsd.edu/NZ_EQ/)

### July 2019 M$_{W}$ 7.1 Ridgecrest, strike-slip faulting

[ALOS-2 and Sentinel-1 interferograms](https://topex.ucsd.edu/SV_7.1/)

### Feb 2023 doublet M$_{W}$ 7.8, 7.5 Turkey, strike-slip faulting, ALOS-2

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
