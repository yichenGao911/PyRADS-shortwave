# PyRADS
PyRADS is the Python line-by-line RADiation model for planetary atmosphereS. PyRADS is a radiation code that can provide line-by-line spectral resolution, yet is written in Python and so is flexible enough to be useful in teaching.

For Earth-like atmospheres, PyRADS currently uses HITRAN 2016 line lists (http://hitran.org/) and the MTCKD continuum model (http://rtweb.aer.com/continuum_frame.html).

This version of PyRADS tackles shortwave calculations using the DISORT radiative solver.

References:

(1) Koll & Cronin, 2018, PNAS, https://doi.org/10.1073/pnas.1809868115.

(2) Koll & Cronin, 2019, The Astrophysical Journal, https://iopscience.iop.org/article/10.3847/1538-4357/ab30c4/meta.

(3) Stamnes et al, 1988, Applied Optics.

# Installation
1) Download to your own computer, or upload to your Linux server.

2) Install the required libraries using conda. Python 3.7 is the most tested
environment for this repository.

```
cd /path/to/PyRADS-shortwave
conda create -n pyrads python=3.7 numpy scipy matplotlib gfortran
conda activate pyrads
```

On recent macOS machines, especially Apple Silicon machines running an
`osx-64` conda environment under Rosetta, it can be useful to install the
compiler packages explicitly:

```
conda install -n pyrads -c conda-forge gfortran_osx-64 clang_osx-64
```

3) Manually compile the MTCKD model:

```
cd /path/to/PyRADS-shortwave/DATA/MT_CKD_continuum/cntnm.H2O_N2/build
make -f make_cntnm osxGNUCONDAdbl
```

Use `osxGNUCONDAdbl` on macOS when using conda's `gfortran`. If you are using a
non-conda Fortran compiler on macOS, try `osxGNUdbl` instead.

On Linux, open `README.build_instructions` in the same directory and choose the
command for your platform. For example, with gfortran:

```
make -f make_cntnm linuxGNUdbl
```

After compiling on Linux, you may need to update the executable name in
`pyrads/Absorption_Continuum_MTCKD.py` to match the generated
`cntnm_v3.2_*` binary.

4) Manually install the pyDISORT wrapper, which solves the radiative transfer
equations with scattering, used for the shortwave calculations.

On recent macOS versions, set the SDK path before building:

```
export SDKROOT=$(xcrun --show-sdk-path)
export CONDA_BUILD_SYSROOT=$SDKROOT
export MACOSX_DEPLOYMENT_TARGET=10.9
export LDFLAGS="-isysroot $SDKROOT"
export CFLAGS="-isysroot $SDKROOT"
```

Then install pyDISORT:

```
cd /path/to/PyRADS-shortwave/PyDISORT3
python setup.py install
```

To test whether pyDISORT was successfully installed:

```
cd /path/to/PyRADS-shortwave/PyDISORT3
python test_disort.py
python test/test_Rayleigh.py
python -c "import disort; print(disort.__file__)"
```

If the installation failed, the test scripts will return an import or dynamic
library error. If the installation worked, the test scripts will print a large
amount of DISORT output.

5) Run test scripts

To compute outgoing longwave radiation (OLR) in W/m2 for a given surface temperature:

```
cd /path/to/PyRADS-shortwave/Test01.olr
python compute_olr_h2o.py
```

To compute OLRs for a set of surface temperatures and save the resulting output to txt:

```
cd /path/to/PyRADS-shortwave/Test02.runaway
python compute_olr_h2o.01.100RH.py
```

To compute SW fluxes in W/m2 for a given surface temperature (here, 300 K) over a *limited* part of the solar spectrum (here, 1000-2000 cm-1) at some resolution (here, 1 cm-1; see note below) and save the resulting output to txt file in the same directory ("."):

```
cd /path/to/PyRADS-shortwave/Test03.sw
python compute_sw_h2o.py 1000. 2000. 1. 300. .
```

To stitch together the SW fluxes across the entire solar spectrum (takes a while even at low spectral res; see note below):

```
cd /path/to/PyRADS-shortwave/Test04.sw_full_spectrum
python compute_sw_h2o.py 1000. 10000. 1. 300. .
python compute_sw_h2o.py 10000. 20000. 1. 300. .
python compute_sw_h2o.py 20000. 30000. 1. 300. .
python compute_sw_h2o.py 30000. 50000. 1. 300. .
python compute_sw_h2o.py 50000. 80000. 1. 300. .
python merge_spectrum.py
```

The `Test04.sw_full_spectrum` directory includes example precomputed spectral
chunks, so `python merge_spectrum.py` can be used as a quick merge test without
rerunning the full spectrum. To verify the full-spectrum compute script without
waiting for all chunks, run a small interval first, for example:

```
cd /path/to/PyRADS-shortwave/Test04.sw_full_spectrum
python compute_sw_h2o.py 1000. 1002. 1. 300. /tmp/pyrads-test04-small
```

NOTE: computing opacities + running pyDISORT over the entire solar spectrum becomes computationally very costly. It is much faster to split the spectral calculations up over many spectral chunks, distribute those over parallel processors, and then combine the spectrally resolved calculations at the end. Use `merge_spectrum.py` or `pyrads/Merge_Spectral_Output.py` to combine discrete chunks of the spectrum.

NOTE: spectral resolution should reflect the available data. E.g., HITRAN2016 doesn't contain H2O lines beyond ~25000 cm-1, so there is no need to retain high spectral resolution in the UV.

NOTE: resolution in test scripts was chosen for relative speed, not accuracy. For research-grade output and model intercomparisons, vertical and spectral resolution need to be increased. For some reference values, see Methods in Koll & Cronin (2018) and Koll & Cronin (2019).


# Requirements
Python 3.7 with numpy and scipy.

For the MTCKD continuum model: gmake and gfortran.

For the pyDISORT radiation solver: see https://github.com/chanGimeno/pyDISORT.
For the Python 3 version of pyDISORT: see https://github.com/danielkoll/PyDISORT3

# Acknowledgements
PyRADS makes use of HITRAN 2016 line lists (http://hitran.org/), AER's MTCKD continuum model (http://rtweb.aer.com/continuum_frame.html), and the PyTran script published by Ray Pierrehumbert as part of the courseware for "Principles of Planetary Climates" (https://geosci.uchicago.edu/~rtp1/PrinciplesPlanetaryClimate/). The SW version of PyRADS uses DISORT, developed by Stamnes et al, and the pyDISORT wrapper, developed by chanGimeno (https://github.com/chanGimeno/pyDISORT). Brian Rose (http://www.atmos.albany.edu/facstaff/brose/), Andrew Williams (https://andrewilwilliams.github.io/), and Zhiping Zhang have improved the code. 
