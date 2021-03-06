# DisVis


## About DisVis

**DisVis** is a Python package and command line tool to visualize and quantify 
the accessible interaction space of distance restrained binary biomolecular complexes.
It performs a full and systematic 6 dimensional search of the three translational
and rotational degrees of freedom to determine the number of complexes consistent
with the restraints. In addition, it outputs the percentage of restraints being violated
and a density that represents the center-of-mass position of the scanning chain corresponding 
to the highest number of consistent restraints at every position in space.


## Requirements

* Python2.7
* NumPy

Optional for faster CPU version

* FFTW3
* pyFFTW

For GPU hardware acceleration the following packages are also required

* OpenCL1.1+
* [pyopencl](https://github.com/pyopencl/pyopencl)
* [clFFT](https://github.com/clMathLibraries/clFFT)
* [gpyfft](https://github.com/geggo/gpyfft)

Recommended for installation

* [git](https://git-scm.com/download)
* [pip](https://pip.pypa.io/en/latest/installing.html)
* Cython


## Installation

If the requirements are met, **DisVis** can be installed by opening a terminal
and typing

    git clone https://github.com/haddocking/disvis.git
    cd disvis
    python setup.py install

The last command might require administrator priviliges for system wide installation.
After installation, the command line tool *disvis* should be at your disposal.

If you are starting from a clean system, then read the installation instructions 
below to prepare your particular operating system.
The INSTALLATION.md file contains instructions for accessing GPU acceleration on MacOSX.


### Linux

First install git and check whether the Python header files and the Python
package manager *pip*, are available by typing

    sudo apt-get install git python-dev python-pip

If you are working on a Fedora system, replace *apt-get* with *yum*.
The final step to prepare you system is installing the Python dependencies

    sudo pip install numpy cython

Wait untill the installation is finished.
Your system is now ready. Follow the general instructions above to install **DisVis**.


### MacOSX

First install [*git*](https://git-scm.com/download) for MacOSX (this might already be present on your system)
by either installing it from the website or using *brew*

    brew install git

Next, install [*pip*](https://pip.pypa.io/en/latest/installing.html), 
the official Python package manager. 
Either follow the link and install *pip* using
their installation instructions or open a terminal and type

    sudo easy_install pip

The final step to prepare your system, is installing the Python dependencies.
In a terminal, type

    sudo pip install numpy cython

Wait till the compilation and installation is finished. 
Your system is now properly prepared. Follow the general instructions above to install **DisVis**.


### Windows

First install [*git*](https://git-scm.com/download) for Windows, as it also comes
with a handy *bash* shell.

For Windows it easiest to install a Python distribution with NumPy and Cython
(and many other) packages included, such as [Anaconda](https://continuum.io/downloads).
Follow the installation instructions on their website.

Next open a *bash* shell that was shipped with *git*. Follow the general instructions
above to install **DisVis**.


## Usage

The general pattern to invoke *disvis* is

    disvis <pdb1> <pdb2> <distance-restraints-file>

where `<pdb1>` is the fixed chain, `<pdb2>` is the scanning chain and 
`<distance-restraints-file>` is a text-file
containing the distance restraints in the following format

     <chainid 1> <resid 1> <atomname 1> <chainid 2> <resid 2> <atomname 2> <mindis> <maxdis>

As an example
    
    A 18 CA F 27 CB 10.0 20.0

This puts a distance restraint between the CA-atom of residue 18 of 
chain A of pdb1 and the CB-atom of residue 27 of chain F of pdb2 that 
should be longer than or equal to 10A and smaller than or equal to 20A.
Comments can be added by starting the line with the pound sign (#) and empty
lines are ignored.


### Options

To get a help screen with available options and explanation type
            
    disvis --help

Some examples to get you started. To perform a 5.27 degree rotational search and store the results
in the directory *results/*

    disvis 1wcm_A.pdb 1wcm_E.pdb restraints.dat -a 5.27 -d results

Note that the directory is created if it doesn't exist.

To perform a 9.72 degree rotational search with 16 processors and a voxel spacing of 2A

    disvis O14250.pdb Q9UT97.pdb restraints.dat -a 9.72 -p 16 -vs 2

To offload computations to the GPU (if the requirements are met) and increase
the maximum allowed volume of clashes as well as the minimum required volume of
interaction, and set the interaction radius to 2A

    disvis 1wcm_A.pdb 1wcm_E.pdb restraints.dat -g -cv 6000 -iv 7000 -ir 2

To perform a grid occupancy analysis for complexes consistent with at least N
restraints, include the `-oa` or `--occupancy-analysis` flag to the command. By
default, the analysis is only performed on complexes consistent with
either *all* and *all - 1* restraints. If you want to perform an occupancy
analysis for complexes consistent with a lower number of restraints, use the
`-ic` or `--interaction-restraints-cutoff` option, followed by the minimum
number of required consistent restraints. Thus the command

    disvis 1wcm_A.pdb 1wcm_E.pdb restraints.dat -oa -ic 5

will make an occupancy grid for complexes consistent with at least 5
restraints, and higher.

Finally, to perform an interaction analysis to determine which residues
are most likely at the interface, the `-is` or `--interaction-selection`
options can be used with an additional file giving the residue numbers of the
receptor and ligand for which the interaction analysis will be performed. The
input file consists of two lines, where the two lines are a space separated
sequence of residue numbers for the receptor and ligand, respectively. As the
interaction analysis is particularly expensive, only complexes consistent with
*all*, or *all - 1* are considered. To lower the barrier, the `-ic` option can
again be used. Also limit the selected residues to only the solvent accessible
ones. A simple example for the selection-file

    1 2 3 4
    101 302 888

This will select residue numbers 1, 2, 3, and 4 for the receptor, and 101, 302,
and 888 for the ligand.


### Output

Standard *disvis* outputs 5 files:

* *accessible_complexes.out*: a text file containing the number of complexes
  consistent with a number of restraints. 
* *violations.out*: a text file showing how often a specific restraint is
  violated for each number of consistent restraints. This helps in identifying
  which restraint is most likely a false-positive if any.
* *accessible_interaction_space.mrc*: a density file in MRC format. The density
  represents the center of mass of the scanning chain conforming to the maximum
  found consistent restraints at every position in space. The density can be
  inspected by opening it together with the fixed chain in a molecular viewer
  (UCSF Chimera is recommended for its easier manipulation of density
  data, but also PyMol works).
* *z-score.out*: a text file giving the Z-score for each restraint. The higher
  the score, the more likely the restraint is a false-positive. Z-scores above
  1.0 are explicitly mentioned in the output.
* *disvis.log*: a log file showing all the parameters used, together with date
  and time indications.

If an occupancy and/or interaction analysis was requested, *disvis* also
outputs the following files:

* *occupancy_N.mrc*: a volume file giving a normalized indication of how
  frequent a grid point is occupied by the ligand, where *N* indicates the minimal
  required number of consistent restraints that resulted in the occupancy grid.
* *[receptor|ligand]_interactions.txt*: a text file containing the average
  number of interactions per complex each selected residue made, for both the
  receptor and ligand molecule. Each column denotes the minimal number of
  consistent restraints of each complex for which interactions were counted.


Licensing
---------

If this software was useful to your research please cite us

**Van Zundert, G.C.P. and Bonvin, A.M.J.J.** (2015) DisVis: Visualizing and
quantifying accessible interaction space of distance restrained biomolecular complexes.
*Bioinformatics 31*, 3222-3224.

Apache License Version 2.0

The elements.py module is licenced under the MIT License, Copyright Christoph
Gohlke.


## Tested platforms

| Operating System| CPU single | CPU multi | GPU |
| --------------- | ---------- | --------- | --- |
|Linux            | Yes        | Yes       | Yes |
|MacOSX           | Yes        | Yes       | No  |
|Windows          | Yes        | Fail      | No  |

The GPU version has been tested on:
NVIDIA GeForce GTX 680 and AMD Radeon HD 7730M for Linux
