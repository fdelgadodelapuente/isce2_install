# ISCE2 installation and Interferogram Processing



Instructions for compiling the software in macOS. These are modified from [Instructions for MacPorts (Piyush Agram, formerly at JPL)](https://github.com/piyushrpt/mojaveSetup) I have used these instrucctions with the following macOS versions and compilers 

ISCE 2.2.0. Aug 2018 MacMini2014/MacBookAir2015, High Sierra and Mojave, gcc7.

ISCE 2.3.1-2.5.2. Feb 2019 MacMini2014/MacBookAir2015, High Sierra and Mojave, gcc7.

ISCE 2.5.3-2.6.3. Nov 2021 MacMini2014, Monterey, python37 and gcc11.

ISCE 2.6.3. Feb 2025 MacBookAir2015, Monterey, python312 and gcc13.

ISCE 2.6.3. Feb 2025 MacBookAir2015, Monterey, python39 and gcc11 after python312 failure in running stripmapApp. Default compiler in the mp system is gcc13, though.

ISCE 2.6.4. Aug 2025 MacMini2024, Sequoia, python313 and gcc13. 

ISCE 2.6.4. May 2026 MacBookNeo, Tahoe, python313 and gcc13.



```


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

```


Create /Applicatons/isce/SConfigISCE file for scons
```
PRJ_SCONS_BUILD   = /Applications/isce/isce2-2.6.4/build/isce
PRJ_SCONS_INSTALL = /Applications/isce/isce2-2.6.4/install/isce


LIBPATH = /opt/local/lib
#the last path in CPP is new for autoRIFT in 2.4.x version
CPPPATH = /opt/local/Library/Frameworks/Python.framework/Versions/3.13/include/python3.13  /opt/local/include /opt/local/include/opencv4  /opt/local/lib/opencv4 /opt/local/Library/Frameworks/Python.framework/Versions/3.13/lib/python3.13/site-packages/numpy/core/include
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

Now install it in Applicatons/isce/isce2-2.6.4
```
rm -rf config.log .sconfig.dblite .sconf_temp .sconsign.dblite; SCONS_CONFIG_DIR=/Applications/isce scons install  
```

Source it with shell script called insar.sh. Inp is the software version (2.2.0, 2.5.1, 2.6.4, etc)
```
#!/bin/sh

inp="$1"
echo "Loading ISCE $1"
 
export PYTHONPATH=/Applications/isce/isce-$1/install:$PYTHONPATH
export PATH=/Applications/isce/isce-$1/install/isce/bin:$PATH
export PATH=/Applications/isce/isce-$1/install/isce/applications:$PATH
export ISCE_HOME=/Applications/isce/isce-$1/install/isce
 
export PATH=/Applications/isce/isce-$1/contrib/stack/stripmapStack:$PATH
#export PATH=/Applications/isce/isce$1/contrib/stack/topsStack:$PATH
  
export GDAL_DATA=/opt/local/share/gdal
```

Patches for ISCE 2.6.4 that are required to run with the latest languages and for the latest missions

[Fix type casting in chi squared print statement in stripmapApp misregistration step](https://github.com/isce-framework/isce2/pull/1021)

[Sentinel-1D patch](https://github.com/isce-framework/isce2/pull/28123)

[SAOCOM-1 stack processor and ALOS-4 single interferogram and stack processor patches](https://github.com/isce-framework/isce2/pull/982)

## Software Manual
[ISCE2 for Earth Science](https://github.com/fdelgadodelapuente/isce2_install/blob/main/tutorial.md)

## Interferogram Processing

You can test the software by processing the following X-, C- and L-band interferograms.

[ENVISAT C-band stripmap, Yellowstone caldera (USA)](https://github.com/fdelgadodelapuente/isce2_install/blob/main/envisat_example.md)

[ENVISAT C-band stripmap, Kilauea caldera (USA)](https://github.com/fdelgadodelapuente/isce2_install/blob/main/envisat_kilauea_example.md)

[ALOS L-band stripmap, Pichilemu earthquake (Chile) ](https://github.com/fdelgadodelapuente/isce2_install/blob/main/alos_example.md)

[ALOS L-band stripmap, Yutian earthquake (China)](https://github.com/fdelgadodelapuente/isce2_install/blob/main/alos_yutian_example.md)

[Sentinel-1 C-band TOPS, Calbuco eruption (Chile) ](https://github.com/fdelgadodelapuente/isce2_install/blob/main/sentinel1_example.md)

[Sentinel-1C/D C-band TOPS, tandem pair, Northern Patagonia Icefield (Chile) ](https://github.com/fdelgadodelapuente/isce2_install/blob/main/sentinel1_patagonia.md)

[ALOS-2 L-band ScanSAR, Aniakchak Crater (USA)](https://github.com/fdelgadodelapuente/isce2_install/blob/main/alos2_example.md)

[SAOCOM-1 L-band stripmap, Santiago basin (Chile)](https://github.com/fdelgadodelapuente/isce2_install/blob/main/saocom1_example.md)

[PAZ X-band stripmap, Salar de Atacama basin (Chile)](https://github.com/fdelgadodelapuente/isce2_install/blob/main/paz_example.md)

[CSK X-band stripmap, Kilauea caldera (USA)](https://github.com/fdelgadodelapuente/isce2_install/blob/main/csk_example.md)


## Stack processor

[TOPS Salar de Atacama](https://github.com/fdelgadodelapuente/isce2_install/blob/main/mintpy_salaratacama.md). This is in spanish.

[TOPS stack processor](https://github.com/fdelgadodelapuente/isce2_install/blob/main/stackproc_tops.md)

[ALOS-2 stack processor](https://github.com/fdelgadodelapuente/isce2_install/blob/main/stackproc_alos2.md)

[Stripmap stack processor](https://github.com/fdelgadodelapuente/isce2_install/blob/main/stackproc_sm.md)
