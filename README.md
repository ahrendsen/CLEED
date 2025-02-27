CLEED package
=============

Introduction
------------

CLEED is a computational package for Low Energy Electron Diffraction (LEED) IV analysis. It fits 
experimental IV data curves with those simulated by the cleed program and a 'goodness' of fit is made
with the R-Factor program crfac. Optimisation of the input parameters is performed by csearch.

A (still rather incomplete) documentation of the program package can be found 
in 'doc/CLEED_Manual.pdf', but it is probably best to send me an email, with all 
input files, to <g.held@reading.ac.uk> when you encounter any problems (I am sure you will ...)

Installation
------------

In order to target as many different systems in the simplest way the CLEED package has been 
developed using CMake as a dependency tracking and makefile generation program. CMake is 
available on all major desktop platforms and a GUI is also available for Windows users.

The dependencies on Linux can be met by installing the following packages
		
    sudo apt install libopenblas-dev ocl-icd-opencl-dev \
    opencl-headers clinfo ocl-icd-libopencl1 gfortran \
    libgsl-dev qt4-dev-tools cmake-qt-gui libtiff-dev
    
Note that if you're using Ubuntu 20.04 or above, Qt4 has been removed from the 
repositories, so qt4-dev-tools won't be found.
I hope to work on updating the program in the future to Qt5, but for now, 
you'll just have to install an earlier version of Ubuntu if you want to run CLEED.
You could attempt installing qt4 on Ubuntu 20.04 as noted 
[here](https://askubuntu.com/questions/1234786/qt4-libqt4-in-ubuntu-20-04),
but I haven't tried this myself so I can't say if it will work, but I wish 
you the best of luck!

I'm going ahead with trying the Qt5 way, since it seems like Liam Deacon
had started the conversion process. The Qt5 packages you need to install
are ([apparently](https://stackoverflow.com/questions/67373586/install-qt5-on-ubuntu-21-04)):

    sudo apt-get install qtbase5-dev qtchooser qt5-qmake qtbase5-dev-tools

There are also python packages which will be used. I'll describe how you 
can install them using Anaconda.

To install anaconda,
=======
    
There are also python packages which will be used. I'm still shaky on how
environments work in python, but I'll assume you use Anaconda, because
I can get it to work with that system.

If you don't already have anaconda installed, and you're just installing 
it to run this program, I would recommend that you install miniconda, 
since it will take up less space, doesn't have unneccesary programs, 
and will install faster. You can find the download link and install 
instructions at
[this webpage](https://docs.conda.io/projects/conda/en/latest/user-guide/install/linux.html):

Make sure to restart your shell after installing it.
You'll know its installed because there will be the string '(base)' 
before your typical command prompt now.

You should make a new conda
environment for running CLEED, otherwise you'll run into dependency/
version issues.

    conda create -n cleed
    conda activate cleed

Then the python programs that are automatically installed shouldn't have
version conflicts with the other python programs on your system.

Use the following commands in order to install CLEED for your system:

    cd /path/to/cleed/source
    export RC=gcc
    cmake-gui

For the "Source" you should select the "CLEED" folder. 
For the "Build" you should select the "CLEED/build" folder. 
Once you have those folders selected, you can press "Configure" 
and then "Generate" and you're ready to proceed to the next step.

Alternatively you can accomplish the same thing in a command line 
interface with the command

    # But I haven't found out the proper syntax for this command.
    # so you're on your own for now.
    cmake -G <generator-name> -D CMAKE_RC_COMPILER=<compiler-name> -B build/ . 

Where <generator-name> is the target platform e.g. "Unix Makefiles" or "MinGW Makefiles". And <compiler-name> is "gcc" for me, but you might want something different. The above command successfully executes, but I only accomplished this after mangling a few other files, the modifications of which should be indicated in this commit.
	
Once the project has been built successfully, there will be a new folder called build.
You can then compile using the following commands.

    cd build/
    <make> install

<make> is usually "make" (Unix/GNU Linux) or "mingw32-make" (Windows). 

Note Windows users will require MinGW installed (other compiler tool chains such as MSVC 
and CYGWIN have not been tested). 

If you are unable to compile the programs, pre-compiled packages are also available.


Getting Started
---------------

On other operating systems check the compatibility of the trigonometric functions
and data type used for real in "real.h" (in directory INCLUDE)

We have had a few problems on OSs different from DEC alpha with the
symmetrised version of CLEED (cleed_sym), therefore this is currently not supported
in this distribution, but the non-symmetrised version should be running okay.

A set of sample input files for Er/Si(111) (no expt. IV curves), Benzene on Ru(0001) 
and O and Cu/Ni(111) can be found in under SIER, RUBENZ, NIO and NICU in the 'examples/'
directory of the CLEED installation.
 
There are three files for each search (in addition to phase shifts and experimental data)
which have to be created by the user:

<bul_file> *.bul
Bulk geometry and non-geometric parameters

<inp_file> *.inp
Specifies start geometry for the search and search parameters (symmetry etc.)

<ctr_file> *.ctr
Specifies correspondence between theoretical and experimental IV curves.

A detailed description of the file format can be found in 'doc/CLEED_Manual.pdf', but it is 
probably easiest to use one of the sample input files and modify it. 
All other files are created by the programs.
The search can be monitored through the *.log file; at each point of the search the 
current best fit geometries and IV curves can be found in *.rmin and *.pmin.

These files are all described in the CLEED manual.

The phase shift input must have the same format as for the VanHove/Tong programs with a 
first line added that contains the number of energies and lmax.  For each type of atoms 
there must be a separate file in the directory 'phase/' called <atom>.phs. The directory 
is passed to the LEED program through the environment variable CLEED_PHASE.

Before starting any program 3 environment variables have to be set:

CLEED_PHASE (see above)
CSEARCH_LEED (path to executable LEED-IV program: normally cleed_nsym)
CSEARCH_RFAC (path to executable R factor program: crfac)

A shell script doing this can be found in 'bin/set_env' (again, the directory path
of the parent directory must be changed in this file). For users with a Python 
interpreter installed, the script 'set_env.py' allows much more sophisticated setups,
for more details see:
    
    set_env.py --help

The search is started by:

    csearch -i <inp_file>

'csearch' is a master program that calls the LEED-IV program (cleed_nsym) and the 
R factor program (crfac) to calculate IV curves and R factors for a given trial 
structure and optimises the geometry parameters of the surface geometry in order to 
minimise the R factor using the simplex method (other search methods are currently tested).

The LEED-IV program can be called outside the search by

    cleed_nsym -i <par_file> -b <bul_file> -o <res_file> > <out_file>

<par_file> is usually created by the search program. It contains only the
positions of the overlayer atoms that are optimised during the search. An example is 
given in 'examples/NIO'. The LEED-IV program produces a lot of control output (written 
to stdout). Therefore it is best to redirect stdout to a file using the '>' command. 
It is best to check the LEED-IV program separately first, after you have made any 
alterations to the input file.

In order to test the R factor program separately, call it by:

    crfac -c <ctr_file> -t <output_file_from_cleed>

On Linux machines, for the search it is recommended to use the 'nohup' command together 
with '&' which runs the search in the background:

    nohup csearch -i <inp_file> > <out_file> 2> <err_file> &

In the NiO111_2x2O example each iteration takes about 90-100s (2.4 GHz Linux PC), the 
final R factor after convergence should be around 0.1322 and it takes around 183
iteration to get there from the start geometry specified in Ni111_2x2O.inp 
(see attached *.log file).

The example in NICU leads to an R factor of 0.0633 after 63 iterations (ca 15s cpu 
time per iteration).

All information necessary to restart the search at the current position is stored in
*.ver. If the search stops for any reason, e.g. because it has reached the limit
of iterations (currently 2000), it can be restarted by:

    nohup csearch -i <inp_file> -v <ver_file> 1> <out_file> 2> <err_file> &

GOOD LUCK

*-- Georg Held --*


