# WEDDELL
Weddell Sea regional NEMO configuration.

The following code was used in this configuration:

svn co http://forge.ipsl.jussieu.fr/nemo/svn/NEMO/trunk@10176

NB This recipe has be written with the ARCHER HPC INTEL environment in mind.

This configuration is currently being used to test the use of hybrid z-sigma coordinates under the ice-sheets more details to follow.

```
# Change to some working directory of choice
export WORK_DIR=path_to_working_directory
if [ ! -d "$$WORK_DIR" ]; then
  mkdir $WORK_DIR
fi
cd $WORK_DIR

# Checkout the NEMO code from the SVN Paris repository 
svn co http://forge.ipsl.jussieu.fr/nemo/svn/NEMO/trunk@10176
cd trunk

# Now change to CONFIG directory
cd cfgs

# Checkout configuration directory structure
git init .
git clone git@github.com:jdha/WEDDELL.git

# Add it to the configuration list
echo "WEDDELL OPA" >> cfgs.txt
```
At this point you can checkout and compile XIOS or use a version you already have. If you're starting from scratch or want to reproduce the original WEDDELL configuration:

```
# Choose an appropriate directory for your XIOS installation
export XIOS_DIR=path_to_checkout_xios
if [ ! -d "$XIOS_DIR" ]; then
  mkdir $XIOS_DIR
fi
cd $XIOS_DIR
svn co http://forge.ipsl.jussieu.fr/ioserver/svn/XIOS/branchs/xios@
cd xios
mv $WORK_DIR/NEMO-shelf/NEMOGCM/CONFIG/AMM7_RECICLE/arch_xios/* ./arch
rm -rf $WORK_DIR/NEMO-shelf/NEMOGCM/CONFIG/AMM7_RECICLE/arch_xios
./make_xios --full --prod --arch XC30_ARCHER --netcdf_lib netcdf4_par --job 4
```

You can fold the ```make_xios``` command into a serial job. NB ```$NETCDF_DIR``` and ```$HDF5_DIR``` must be part of your environment. This should be the case if you've used ```modules``` to setup the netcdf and hdf5 e.g. 

```
module swap PrgEnv-cray PrgEnv-intel
module load cray-hdf5-parallel
module load cray-netcdf-hdf5parallel
```

Next, compile the NEMO code itself. First we copy the arch files into the appropriate directory.

```
cd $WORK_DIR/NEMO-shelf/NEMOGCM/CONFIG/AMM7_RECICLE/
mv ARCH/* ../ARCH
rm -rf ARCH
```

NB you will either have to edit ```../../ARCH/arch-XC_ARCHER_INTEL_XIOS1.fcm``` replacing ```$XIOS_DIR``` with the expanded ```$XIOS_DIR/xios-1.0``` or define ```$XIOS_DIR``` in your environment.

```
cd ../
./makenemo -n WEDDELL -m XC_ARCHER_INTEL -j 4
```

That should be enough to produce a valid executable. Now to copy the forcing data from JASMIN. 

```
cd WEDDELL/EXP00
wget http://opendap4gws.jasmin.ac.uk/thredds/noc_msm/fileServer/weddell/domain_cfg_zps_new.nc
wget http://opendap4gws.jasmin.ac.uk/thredds/noc_msm/fileServer/weddell/domain_cfg_zps_old.nc
wget http://opendap4gws.jasmin.ac.uk/thredds/noc_msm/fileServer/weddell/domain_cfg_szt_05m.nc
wget http://opendap4gws.jasmin.ac.uk/thredds/noc_msm/fileServer/weddell/domain_cfg_szt_10m.nc
```

And finally link the XIOS binary to the configuration directory.

```
cd ../ENSEMBLE_CONTROL
rm xios_server.exe
ln -s $TO_XIOS_DIR/xios-1.0/bin/xios_server.exe xios_server.exe
```

Edit and run the ```run_script.pbs``` script in ```../EXP00``` accordingly.
