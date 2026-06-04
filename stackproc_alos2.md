
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

If the correction was successful, you need to uncomment the lines that run `ion_ls.py` and `ion_correct.py` and run them in a separate script.

```
/home/fdelgado/isce/isce2-2.5.3/contrib/stack/alosStack/ion_ls.py -idir pairs_ion -odir dates_ion -ref_date_stack 171201 -nrlks1 2 -nalks1 4 -nrlks2 4 -nalks2 4 -nrlks_ion 16 -nalks_ion 16 -interp

insarpair=($(ls -l pairs | grep ^d | awk '{print $9}'))
for ((i=0;i<${#insarpair[@]};i++)); do

  IFS='-' read -ra dates <<< "${insarpair[i]}"
  ref_date=${dates[0]}
  sec_date=${dates[1]}

  cd pairs
  cd ${insarpair[i]}
  #uncomment to run this command
  /home/fdelgado/isce/isce2-2.5.3/contrib/stack/alosStack/ion_correct.py   -ion_dir ../../dates_ion -ref_date ${ref_date}   -sec_date ${sec_date} -nrlks1 2 -nalks1 4 -nrlks2 4   -nalks2 4
  cd ../../

done

```

The specific parameters and file paths depend upon each data set.

Afterwards filter, unwrap and geocode as in the rest of the workflows.

```
./cmd_4.sh
```

