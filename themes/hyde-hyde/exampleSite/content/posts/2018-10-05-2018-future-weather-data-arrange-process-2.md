---
title: "2018-future weather data arrange process_2"
author: "Weilu Wang"
date: '2018-10-05'
highlight: no
lastmod: '2018-10-05T14:11:31+08:00'
layout: post
slug: 2018-future-weather-data-arrange-process-2
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

In this tutorial, we will further introduce how to extracting future weather data with python language.
Thanks vice-professor Yang for his help in translating R command into Python. Thanks Teacher Liu for his help in LINUX system operation.

### File preparation

1. Install Python3.6 in LINUX platform
2. Locate the meteolib file in Lib directory of Python.

### Extracting data

The commands are listed following

```
# -*- coding: utf-8 -*-
"""
Created on Tue Sep 11 13:03:08 2018

@author: yangsbin
"""
#python ./process_nc4_script.py
#export PATH="/public/software/miniconda3/bin:$PATH"
#export HDF5_USE_FILE_LOCKING=FALSE
import datetime as dt
import numpy as np
import scipy as sp
import netCDF4
import matplotlib.pyplot as plt
import pandas as pd
import meteolib
from netCDF4 import Dataset

def fromncs(nc_fid, lat, lon, sy, ey):
    '''
    used to read a nc4 file, and extract data at (lat, lon) for a period of time
    the time span is from sy to ey
    it returns the data
    usage: data = fromncs(nc_fid, 118.12, 120.5, 2001, 2030)
    input:
        nc_fid: fid of a nc4 format data file
        lat: latitude, in decimal degree
        lon: longitude, in decimal degree
        sy: start year
        ey: end year
    output:
        data, a matrix with 30 rows and two columns
    '''
    data = []
    return data

def ncdump(nc_fid, verb=True):
    '''
    ncdump outputs dimensions, variables and their attribute information.
    The information is similar to that of NCAR's ncdump utility.
    ncdump requires a valid instance of Dataset.

    Parameters
    ----------
    nc_fid : netCDF4.Dataset
        A netCDF4 dateset object
    verb : Boolean
        whether or not nc_attrs, nc_dims, and nc_vars are printed

    Returns
    -------
    nc_attrs : list
        A Python list of the NetCDF file global attributes
    nc_dims : list
        A Python list of the NetCDF file dimensions
    nc_vars : list
        A Python list of the NetCDF file variables
    '''
    def print_ncattr(key):
        """
        Prints the NetCDF file attributes for a given key

        Parameters
        ----------
        key : unicode
            a valid netCDF4.Dataset.variables key
        """
        try:
            print("\t\ttype:", repr(nc_fid.variables[key].dtype))
            for ncattr in nc_fid.variables[key].ncattrs():
                print('\t\t%s:' % ncattr,\
                      repr(nc_fid.variables[key].getncattr(ncattr)))
        except KeyError:
            print("\t\tWARNING: %s does not contain variable attributes" % key)

    # NetCDF global attributes
    nc_attrs = nc_fid.ncattrs()
    if verb:
        print("NetCDF Global Attributes:")
        for nc_attr in nc_attrs:
            print('\t%s:' % nc_attr, repr(nc_fid.getncattr(nc_attr)))
    nc_dims = [dim for dim in nc_fid.dimensions]  # list of nc dimensions
    # Dimension shape information.
    if verb:
        print("NetCDF dimension information:")
        for dim in nc_dims:
            print("\tName:", dim )
            print("\t\tsize:", len(nc_fid.dimensions[dim]))
            print_ncattr(dim)
    # Variable information.
    nc_vars = [var for var in nc_fid.variables]  # list of nc variables
    if verb:
        print("NetCDF variable information:")
        for var in nc_vars:
            if var not in nc_dims:
                print('\tName:', var)
                print("\t\tdimensions:", nc_fid.variables[var].dimensions)
                print("\t\tsize:", nc_fid.variables[var].size)
                print_ncattr(var)
    return nc_attrs, nc_dims, nc_vars

def get_data(nc_fid, meteo_type, latx, lony):
    '''
    nc_fid: nc file id
    latx: latitude in decimal degree
    lony: longitude in decimal degree
    output: 
    d_year, the year
    d_month, the month
    d_day, the day
    d_doy, the day of the year, the first day is 1/1
    d_xy, time series of rsds for one grid
    '''
    
    if meteo_type == 'rsds':
        meteo = nc_fid.variables[meteo_type][:]
        meteo_unit = 'W m-2'
    if meteo_type == 'pr':
        meteo = nc_fid.variables[meteo_type][:]
        meteo_unit = 'kg m-2 s-1'
    if meteo_type == 'tasmax' or meteo_type == 'tasmin' or meteo_type == 'tas':
        meteo = nc_fid.variables[meteo_type][:]
        meteo_unit = 'K'
    if meteo_type == 'wind':
        meteo = nc_fid.variables[meteo_type][:]
        meteo_unit = 'm s-1'
    if meteo_type == 'rhs':
        meteo = nc_fid.variables[meteo_type][:]
        meteo_unit = '%'
    
    # extract/copy the data
    lats = nc_fid.variables['lat'][:]  
    lons = nc_fid.variables['lon'][:]
    time = nc_fid.variables['time'][:]
    time_unit = 'days since 1860-01-01 00:00:00'
    time_calendar = 'standard'
    
    time_f = netCDF4.num2date(time, time_unit, time_calendar)  #change the date format
    
    # only for north hemisphere and 0-180E
    if np.round(latx) > latx:
        gridx = int((90-np.round(latx))*2)
    else:
        gridx = int((90-np.round(latx))*2 + 1)
    if np.round(lony) > lony:
        gridy = int(180*2 + np.round(lony)*2)
    else:
        gridy = int(180*2 + np.round(lony)*2 + 1)

    d_xy = list(range(len(time)))
    d_year = list(range(len(time)))
    d_month = list(range(len(time)))
    d_day = list(range(len(time)))
    d_doy = list(range(len(time)))
    for i in range(0,len(time)):
        d_xy[i] = meteo[i][gridx-1,gridy-1]  # one grid data with coordinate (latx, lonx)
        d_year[i] = time_f[i].year
        d_month[i] = time_f[i].month
        d_day[i] = time_f[i].day
        d_init = dt.datetime(d_year[i], 1, 1)
        d_doy[i] = (time_f[i] - d_init).days + 1
        #print(d_xy[i])

    return d_year, d_month, d_day, d_doy, d_xy #year, month, day, doy, value

#example point
lony = 118.3  #118.3E
latx = 30.2   #30.2N

# RSDS DATA
nc_f = 'D:\\rsds_bced_1960_1999_noresm1-m_rcp4p5_2006-2010.nc4'  #your filename and path
nc_fid = Dataset(nc_f, 'r',format='NETCDF4')
nc_attrs, nc_dims, nc_vars = ncdump(nc_fid)
rs = np.array(get_data(nc_fid, 'rsds', latx,lony))
nc_fid.close()
 
# PRECIPITATION DATA
nc_f = 'D:\\pr_bced_1960_1999_noresm1-m_rcp4p5_2006-2010.nc4'  #your filename and path
nc_fid = Dataset(nc_f, 'r',format='NETCDF4')
nc_attrs, nc_dims, nc_vars = ncdump(nc_fid)
pr = np.array(get_data(nc_fid, 'pr', latx,lony))
nc_fid.close()

# TMAX DATA
nc_f = 'D:\\tasmax_bced_1960_1999_noresm1-m_rcp4p5_2006-2010.nc4'  #your filename and path
nc_fid = Dataset(nc_f, 'r',format='NETCDF4')
nc_attrs, nc_dims, nc_vars = ncdump(nc_fid)
tasmax = np.array(get_data(nc_fid, 'tasmax', latx,lony))
tasmax[-1] = tasmax[-1] - 273.15
nc_fid.close()

# TMIN DATA
nc_f = 'D:\\tasmin_bced_1960_1999_noresm1-m_rcp4p5_2006-2010.nc4'  #your filename and path
nc_fid = Dataset(nc_f, 'r',format='NETCDF4')
nc_attrs, nc_dims, nc_vars = ncdump(nc_fid)
tasmin = np.array(get_data(nc_fid, 'tasmin', latx,lony))
tasmin[-1] = tasmin[-1] - 273.15
nc_fid.close()

# TAVG DATA
nc_f = 'D:\\tas_bced_1960_1999_noresm1-m_rcp4p5_2006-2010.nc4'  #your filename and path
nc_fid = Dataset(nc_f, 'r',format='NETCDF4')
nc_attrs, nc_dims, nc_vars = ncdump(nc_fid)
tas = np.array(get_data(nc_fid, 'tas', latx,lony))
tas[-1] = tas[-1] - 273.15
nc_fid.close()

# WIND DATA
nc_f = 'D:\\wind_bced_1960_1999_noresm1-m_rcp4p5_2006-2010.nc4'  #your filename and path
nc_fid = Dataset(nc_f, 'r',format='NETCDF4')
nc_attrs, nc_dims, nc_vars = ncdump(nc_fid)
wind = np.array(get_data(nc_fid, 'wind', latx,lony))
nc_fid.close()

# RHS DATA
nc_f = 'D:\\rhs_noresm1-m_rcp4p5_2006-2010.nc4'  #your filename and path
nc_fid = Dataset(nc_f, 'r',format='NETCDF4')
nc_attrs, nc_dims, nc_vars = ncdump(nc_fid)
rhs = np.array(get_data(nc_fid, 'rhs', latx, lony))
nc_fid.close()

# VAPOR PRESSURE
sat_vpr = meteolib.es_calc(tas[-1])  #unit: Pa
vpr = sat_vpr*rhs[-1]/100.0/1000.0   #unit:kPa

# convert array into a dataframe
output_data = np.vstack((rs[0], rs[1], rs[2], rs[3], rs[4]*86400.0/1000.0, tasmax[-1], tasmin[-1], vpr, wind[-1], pr[-1]*86400.0)) #output data
df = pd.DataFrame(np.transpose(output_data))  #into a dataframe
filepath = 'D:\\my_excel_file.xlsx'  # save to xlsx file
df.to_excel(filepath, index=False, header=['Year', \
                                           'Month',\
                                           'Day',\
                                           'DOY',\
                                           'RAD(kJ/m2/d)',\
                                           'TMAX(oC)',\
                                           'TMIN(oC)',\
                                           'VPR(kPa)',\
                                           'WIND(m/s)',\
                                           'PR(mm/d)'], float_format='%.1f')
                                           
```
You just have to change
lony = 118.3  #118.3E
latx = 30.2   #30.2N




