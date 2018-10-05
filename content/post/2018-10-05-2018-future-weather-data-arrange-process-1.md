---
title: "2018-Future weather data arrange process_1"
author: "Weilu Wang"
date: '2018-10-05'
output:
  html_document:
    df_print: paged
  pdf_document: default
lastmod: '2018-10-05T13:31:50+08:00'
layout: post
highlight: no
slug: 2018-future-weather-data-arrange-process-1
tags:
- R
- Learning exprience
- Weather data
categories:
- R
- Data management
type: post
---

**Insert Lead paragraph here.**

To process future weather data simulated by five CMIP models, I contacted with teacher Yang, who is a vice-professor  and working at **Nanjing Universtiy of Information Science and Technology**. This is the first time for us to cooperate with each other. Thanks for his expertised work on the programm related to future weather data process. I have extracted the corresponding data from five different results simulated by different CMIP model. I will list the procedure and commands. 

### Extracting weather data from CMIP database

The following commands are the details accounting for the future data process on LINUX system platform.

1. Install CDO packages
You can directly install software by yourseld if you have own a server, otherwise contact the adminstrator.
  
export PATH=/public/software/anaconda2/bin:$PATH  #Add path for the software
export HDF5_USE_FILE_LOCKING=FALSE  #Need for multithreaded operations  

2. File infromation and manipulation

Use the command lines to check the file or manipulate it.

```
#CDO DOES NOT HAVE a graphic environment! It only processes data.
#The commands run directly on the Linux terminal.
Note: For NetCDF data support, the installation of the NetCDF libraries are required;
Syntax: cdo <operator> input.nc output.nc or
cdo <operator> input.nc.

#File Information:
1 infon: information about the dataset listed by the variable name.
2 sinfon: information on the dataset in a reduced form.
3 griddes: shows information about the ﬁle domain.
4 zaxisdes: shows information about the vertical coordinate.
5 pardes: description of the variable or variables of the ﬁle.

cdo infon standard.nc | head
cdo infov ifile 
cdo showtimestamp ~
cdo sellonlatbox  ~

#File Manipulation:
1 copy: duplicate the ﬁle and/or convert to another format.
2 cat: increases the number of ﬁle times.
3 merge: joins ﬁles with diﬀerent variables.
4 split: divide the ﬁle (vertical level or time).

copy: duplicates the ﬁle and/or converts from one format to another.
#Note:The ﬁles must have the same dimensions (latitude, longitude, vertical levels).

Example1: Converts from NetCDF to GRIB:
cdo -f grb copy pr.GPCP.1996.2005.nc pr.GPCP.1996.2005.grb
Example2: Converts from GRIB to NetCDF:
cdo -f nc copy pr.GPCP.1996.2005.grb pr.GPCP.1996.2005.nc
Example of concatenation with the copy operator:
cdo copy pr.GPCP.1996.1999.nc pr.GPCP.2000.2002.nc pr.1996.2002.nc


cat: increases the number of ﬁle times.
cdo cat pr.GPCP.1996.1999.nc pr.GPCP.2000.2002.nc pr.1996.2002.nc

merge: joins ﬁles with diﬀerent variables.
cdo merge u.NCEP.R2.1996.2005.nc v.NCEP.R2.1996.2005.nc uv.nc
Note:Files do not necessarily have to have the same dimensions (latitude, longitude, vertical levels).


split: divide or separate the ﬁle.
Operator: splitlevel, splithour, splitday, splitmon, splitseas and splityear
Example: Separate ﬁle in months:
cdo splitmon pr.GPCP.1996.2005.nc month.

#Where: month. represents a preﬁx or any name deﬁned by user and pr.GPCP.1996.2005.nc is the input ﬁle.
#There will be 12 ﬁles created: month.01.nc, month.02.nc, ... ,
#month.12.nc.


Selection Operators:
1 sellevel: selects the vertical level.
2 selmon: selects the month of the ﬁle.
3 selseason: selects the season of the year.
4 selyear: selects the year ﬁle.
5 sellonlatbox: cut the data in the domain of interest

cdo selyear,2000/2002 tar.NCEP.R2.1996.2005.nc tar.2000.2002.nc
cdo selyear,1996/1998,2000 tar.NCEP.R2.1996.2005.nc tar.1996a1998.2000.nc
cdo seldate,2014-12-12T12:00:00, 2015-01-31T18:00:00 infile.nc outfile.nc 

sellonlatbox: cut the data according to the area of interest.
cdo sellonlatbox,-90,-30,-60,10 pr.GPCP.1996.1999.nc pr.AS.nc
#The convention will always be: longitude west, longitude east, latitude south and north latitude
#Longitude values can be reported in either the 0 to 360 format and in the -180 to 180 format.

How to convert the global data to -180 to 180? Only works with global data.
cdo sellonlatbox,-180,180,-90,90 pr.GPCP.1996.1999.nc pr.global.nc


setcalendar: sets the calendar type of the ﬁle.
Possibilities: standard, proleptic_gregorian, 360_day, 365_day e366_day.

ncdump -h stan.nc

cdo setcalendar,365_day stan.nc st.nc

setmissval: sets a new missing value (UNDEF).
cdo setmissval,-999 tar.GFDL-ESM2M.1996.2005.nc tar.GFDL-ESM2M.nc


Operations with constants:
1 addc: sum to a constant.
2 subc: subtracts from a constant.
3 mulc: multiplies by a constant.
4 divc: divided by a constant.

乘法

Converting the precipitation of [kg/(m2 s)] to (mm/day).
cdo mulc,86400 pr.CANESM2.1996.2005.nc pr.mensal.nc

减法

Converting Kelvin temperature (K) to degrees Celsius (oC)
cdo subc,273.15 tar.NCEP.R2.1996.2005.nc tc.nc


Operations using two sets of data (uni or two-dimensional).
1 add: adds two ﬁelds.
2 sub: subtract two ﬁelds.
3 mul: multiplies two ﬁelds.
4 div: divide dois campos.

The ﬁles must have the same dimensions (latitude, longitude, vertical levels).
cdo sub pr.CANESM2.2005.nc pr.GPCP.2005.nc dif.nc


Annual statistical calculation. Converts the data to annual time scale
independent of the temporal resolution.
For example, if the data is hourly, daily or monthly, it will be converted
for an annual archive.
1 yearsum: annual sum.
2 yearmean: annual average.
3 yearavg: annual average.

Seasonal statistical calculation::
1 seassum: seasonal sum.
2 seasmean: seasonal average.
3 seasavg: seasonal average.

Statistical calculation - converts to monthly scale..
1 ymonsum: monthly sum.
2 ymonmean: monthly average.
3 ymonavg: monthly average.


Operators to calculate correlation.
ﬂdcor ⇒ correlates all grid points for each time of the two
variables. The result is a time series of correlation between them.
timcor ⇒ performs the correlation at each grid point for all times
of the two variables. The result is a ﬁle spatial correlation with
only one time.

Example1: Correlation between precipitation and temperature. The
data are monthly referring to the year of 2004 (12 months or times).
cdo ﬂdcor prec.mensal.nc temp.mensal.nc output.nc


Examples of other applications:
1 Calculate the anomaly.
2 Create masking.
3 Filling missing data.
4 Change NetCDF ﬁle values.
5 Text ﬁle conversion to NetCDF.
6 Conversion of a radiosonde to the NetCDF format.
```

