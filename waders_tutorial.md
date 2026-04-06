WADERS Installation Guide
======

In this guide, I will walk through my process for installing the WADERS software (written by Nuria Cstello-Mor for the DAMIC-M experiment).

You will need access to the DAMIC-M gitlab https://gitlab.in2p3.fr/  
If you do not have access already, email Nuria at castello@ifca.unican.es to request access.  

A detailed Wiki for the pysimdamicm (WADERS) software is available https://gitlab.in2p3.fr/damicm/pysimdamicm/-/wikis/home  

Most importantly, root must be installed with a version of python compatible with WADERS. WADERS has been tested on root v6.26 with python 3.9 and v6.30 with python 3.10. In this installation guide, we install WADERS in a conda environment called waders-env with root 6.30 and python 3.10 and other packages. Navigate to a directory you would like to install WADERS into. I recommend installing into a projects directory in your base, such as ~/projects/ on Mac.

The easiest way I have found to install WADERS is by using an environment.yml file to configure a conda environment with the correct python and root versions. Create a file called environment.yml and fill it with the file contents by running:
```
cat > environment.yml << EOF
name: waders-env
channels:
  - conda-forge
  - defaults
dependencies:
  - python=3.10.19
  - root=6.30.4
  - root_base=6.30.4
EOF
  ```

From the directory containing environment.yml, run the following to create the environment and activate it:
```
conda env create -f environment.yml \
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

Now we will install the pysimdamicm (WADERS) package. Navigate to your desired package installation directory (e.g. ~/projects). 
```
cd ~/projects
```

Clone into the pysimdamicm repository on the DAMIC-M Gitlab and checkout the main stable branch waders_LBC
```
git clone https://gitlab.in2p3.fr/damicm/pysimdamicm.git \
git checkout waders_LBC
```

Install dependencies for python 3.10 through requirements.txt file, which includes packages such as numpy, matplotlib, scipy, etc:
```
cd pysimdamicm \
pip install -r requirements_py310.txt
```

Install pysimdamicm package in development mode:
```
pip install -e .
```
Now, if you are on a Mac made after 2020 (with an M1/.../M5 chip), I have run into architecture compatibility issues with certain hard coded numpy precision floats. Essentially, Mac chips do not allow floats with precision greater than 64 bits, so using np.float128 will error. The workaround is to change all instances of np.float128 to np. Add this line to `__init__.py` (in DAMICM_G4Sims/DAMICMSims/python_script/pysimdamicm/pysimdamicm):
```
import numpy as np
if not hasattr(np, "float128"):
    np.float128 = np.float64
```
After you do this, you need to uninstall `psutil` and reinstall it to make sure there are no duplicate paths:
```
pip uninstall -y psutil \
pip install psutil
```

Now you should be good to go! Let's test the analysis software to make sure the main commands run as expected.

There are two main branches of WADERS, `psimulCCDimg` for Geant4 simulation and `panaSKImg` for analysis. The rest of this tutorial will focus only on the analysis branch. Documentation for `psimulCCDimg` is available at https://ncastell.web.cern.ch/ncastell/pysimdamicm/howtouse.html  

We will do our first example using `panaSKImg`. Navigate to the directory you want to do your analysis in. Save the json file `panaSKImg_config_LBC_ACM_4DQM_PA08_103.json` and the image avg_img_CV_250x3500x500_bin1x1_125_20260404_144853_0.fz from this repo, then run:
```
panaSKImg -j panaSKImg_config_LBC_ACM_4DQM_PA08_103.json -o . "avg_img_CV_250x3500x500_bin1x1_125_20260404_144853_0.fz" --save-plots
```

Your Terminal should start printing out status updates for the processes, such as the parameters specified in the json file, which processes it is running, the values for single electron resolution found for each extension, and so on. When that finishes running, look in your directory for plot outputs and admire the beautiful fits!

If the command does not run, it is possible that there are some missing packages. Make sure you are in the waders-env conda environment. If you are in waders-env and there are still import errors, try installing them using conda and running again until it does not error.




