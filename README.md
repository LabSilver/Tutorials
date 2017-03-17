# Tutorials
These guides are to be executed in Orchestra with the BioGrids environment. Initialize BioGrids on your Orchestra account by running:

``$ /programs/biogrids_setup``

You will be asked if you want to modify your startup files. If you type y, you will be able to access BioGrids applications in the future without additional setup. If you type n, your startup files will not be modified, and you will only have access to BioGrids applications in the current shell.
 
More information on BioGrids setup can be found here: https://wiki.med.harvard.edu/Orchestra/BioGrids
Please contact help@biogrids.org if you run into any problems.

Additionally, before we get started, your life will be made much simpler if you do the following:
```
$ echo "alias cleanpdb='python /programs/x86_64-linux/rosetta/3.7/tools/protein_tools/scripts/clean_pdb.py'" >> ~/.bashrc
$ echo "alias rosetta-relax='/programs/x86_64-linux/rosetta/3.7/main/source/bin/relax.linuxgccrelease'" >> ~/.bashrc
```
Normally, we'd then ``source .bashrc`` but BioGrids prevents us from doing this. Instead, ``exit`` and log back into Orchestra.

## Rosetta
In modest attempts at protein redesign we are often concerned with evaluating a handful of rational point mutations for several consequences: impact on protein stability, modification to ligand binding affinity or specificity, and influence on protein-protein interactions. Here, we will use redesign of the corticosteroid binding protein (CBG) as an introduction to Rosetta's applications for informing our choice of point mutants.

You can find any of the requisite files for these tutorials in my folders:
~~~~
$ cd /home/njr8/public/tutorials/
~~~~
(note: Beginning a file address with './' will begin the address from the folder you are currently in. Beginning with '../' will begin the address from *one folder back* from the current folder. Beginning with '/' will start from the root directory. The other user accounts on orchestra can thus be accessed with '/home/{username}')

### Preparing protein structures
Before we can get started in full, we need to obtain a pdb structure of the protein in question.

Ideally, we can just clean the junk crystallization molecules off of a known structure\*:
~~~~
$ cleanpdb {your xtal} {chainIDs}
~~~~

This script will only keep the essential atoms that are used by Rosetta, and convert non-canonical amino acids into the nearest canonical counterpart. After cleaning the pdb, it's important that we "relax" it, so that resulting thermodynamic scores are comparable to the structures we will generate later (such as point mutants).

First, we'll want to create a "flags" file that tells Rosetta what sorts of limitations we want to impose in the relaxation. Enter:
~~~~
$ vi firstrelax.flags
~~~~
Then pressing 'ctrl+I' to write to the new file, paste (middle mouse button, or shift+insert) the following into the file:
~~~~
-
-ex1                # extra rotamer angles 1
-ex2                # extra rotamer angles 2
-use_input_sc       # begin with sidechain angles from a starting input structure
-flip_HNQ           # include alternate protonation states of H, N, and Q
-no_optH false      # allows optimization of hydrogens at the end of the run

~~~~

~~~~
relax.linuxgccrelease  -database Rosetta/main/database -relax:constrain_relax_to_start_coords -relax:coord_constrain_sidechains -relax:ramp_constraints false -s your_structure.pdb
~~~~



For more information, see the RosettaCommons page describing how to prepare structures for Rosetta: https://www.rosettacommons.org/docs/latest/rosetta_basics/preparation/preparing-structures






\* In instances where you don't have a corresponding crystal structure, you'll need to approximate or predict one. See our section on 'structure prediction', coming soon.
### Evaluating the energy cost of a point mutation


### Energy cost of mutations on protein-ligand binding

### Energy cost of mutations on protein-protein binding

