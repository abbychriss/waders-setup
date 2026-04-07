# WADERS Installation Guide

## Introduction

In this guide, I will walk through my process for installing the WADERS software (written by Nuria Cstello-Mor for the DAMIC-M experiment).

You will need access to the DAMIC-M gitlab https://gitlab.in2p3.fr/  
If you do not have access already, email Nuria at castello@ifca.unican.es to request access.  

A detailed Wiki for the pysimdamicm (WADERS) software is available https://gitlab.in2p3.fr/damicm/pysimdamicm/-/wikis/home  

Most importantly, ROOT must be installed with a version of Python compatible with WADERS. WADERS has been tested on ROOT v6.26 with Python 3.9 and v6.30 with Python 3.10. In this installation guide, we install WADERS in a conda environment called waders-env with ROOT 6.30 and Python 3.10 and other packages. Navigate to a directory you would like to install WADERS into. I recommend installing into a projects directory in your base, such as ~/projects/ on Mac.

## Create conda environment

The easiest way I have found to install WADERS is by using an `environment.yml` file to configure a conda environment with the correct versions of Python and ROOT. Using the environment.yml file provided in this repo, run the following to create the conda environment and activate it:
```
conda env create -f environment.yml
conda activate waders-env
```

Before installing WADERS, make sure git is installed. Run the following in terminal
```
git version
```
If you are on a Mac and this errors, check if command line tools are installed by running 
```
xcode-select -p
```
If this outputs a path like `/Library/Developer/CommandLineTools` or `/Applications/Xcode.app/Contents/Developer` then the tools are installed but git is not, so run
```
brew install git
```
If a path to XCode does not print and the command errors, then install the command line tools via
```
xcode-select --install
```
and follow the system prompts to install the library.

## Install WADERS

Now we will install the pysimdamicm (WADERS) package. Navigate to your desired package installation directory (e.g. ~/projects). 
```
cd ~/projects
```

Clone into the pysimdamicm repository on the DAMIC-M Gitlab and checkout the main stable branch waders_LBC
```
git clone https://gitlab.in2p3.fr/damicm/pysimdamicm.git
git checkout waders_LBC
```

Install dependencies for Python 3.10 through requirements.txt file, which includes packages such as numpy, matplotlib, scipy, etc:
```
cd pysimdamicm
pip install -r requirements_py310.txt
```

Install pysimdamicm package in development mode:
```
pip install -e .
```
The -e flag makes the files editable.

Now, if you are on a Mac made after 2020 (with an M1/.../M5 chip), I have run into architecture compatibility issues with certain hard coded numpy precision floats. Essentially, Mac chips do not allow floats with precision greater than 64 bits, so using np.float128 will error. The workaround is to change all instances of np.float128 to np. Add this line to `__init__.py` (in DAMICM_G4Sims/DAMICMSims/python_script/pysimdamicm/pysimdamicm) before the import statements:
```
import numpy as np
if not hasattr(np, "float128"):
    np.float128 = np.float64
```
After you do this, you need to uninstall `psutil` and reinstall it to make sure there are no duplicate paths:
```
pip uninstall -y psutil
pip install psutil
```

## Test installation

Now you should be good to go! Let's test the analysis software to make sure the main commands run as expected.

There are two main branches of WADERS, `psimulCCDimg` for Geant4 simulation and `panaSKImg` for analysis. The rest of this tutorial will focus only on the analysis branch. Documentation for `psimulCCDimg` is available at https://ncastell.web.cern.ch/ncastell/pysimdamicm/howtouse.html  

We will do our first example using `panaSKImg`. Navigate to the directory you want to do your analysis in. Save the json file `panaSKImg_config_LBC_ACM_4DQM_PA08_103.json` and the image avg_img_CV_250x3500x500_bin1x1_125_20260404_144853_0.fz from this repo, then run:
```
panaSKImg -j panaSKImg_config_LBC_ACM_4DQM_PA08_103.json -o . "avg_img_CV_250x3500x500_bin1x1_125_20260404_144853_0.fz" --save-plots
```

If the installation was successful, your Terminal should start outputting a bunch of printouts such as the parameters specified in the json file, which processes it is running, the values for single electron resolution found for each extension, and so on. When that finishes running, look in your directory for plot outputs and admire the beautiful zeroth and first electron peak fits!

### Syntax

The general syntax for `panaSKImg` is
```
panaSKImg -j <path/to/json_file> -o <path/to/output_directory> "<path/to/image.fz/fits>"`
``` 

- The image name as well as flags `-j` and `-o` are required.
- Accepted image format is a multi-extension fits file with ext 0 as header and ext 1-4 as data extensions 

## Troubleshooting

If `panaSKImg` fails and throws a "module not found" error, you may need to install the missing packages. Make sure you have `waders-env` activated (you should see `(waders-env)` at the very beginning of your command line prompt if you are in the environment). 

If `waders-env` is activated and there are still import errors, try installing them using conda and running again until it does not error.

If the command fails with error
```
zsh: command not found: panaSKImg
```
then try pip installing the package again and restarting the Terminal.


