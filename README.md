# Tutorials
These guides are to be executed in Orchestra with the BioGrids environment. Initialize BioGrids on your Orchestra account by running:
'$ /programs/biogrids_setup'
 
You will be asked if you want to modify your startup files. If you type y, you will be able to access BioGrids applications in the future without additional setup. If you type n, your startup files will not be modified, and you will only have access to BioGrids applications in the current shell.
 
More information on BioGrids setup can be found here: https://wiki.med.harvard.edu/Orchestra/BioGrids
Please contact help@biogrids.org if you run into any problems.


## Rosetta
In modest attempts at protein redesign we are often concerned with evaluating a handful of rational point mutations for several consequences: impact on protein stability, modification to ligand binding affinity or specificity, and influence on protein-protein interactions. Here, we will use redesign of the corticosteroid binding protein (CBG) as an introduction to Rosetta's applications for informing our choice of point mutants.

### Preparing protein structures
Before we can get started in full, we need to obtain a pdb structure of the protein in question.

Ideally, we can just clean the junk crystallization molecules off of a known structure\*:
'clean_pdb.py {your xtal} {chainIDs}'

You can obtain 'clean_pdb.py from my folder and copy it into your own folder 'cp /home/njr8/public/tools/clean_pdb.py ./'
(note: Beginning a file address with './' will begin the address from the folder you are currently in. Beginning with '../' will begin the address from *one folder back* from the current folder. Beginning with '/' will start from the root directory. The other user accounts on orchestra can thus be accessed with '/home/{username}')

This script will only keep the essential atoms that are used by Rosetta, and convert non-canonical amino acids into the nearest canonical counterpart. After cleaning the pdb, it's important that we "relax" it, so that resulting thermodynamic scores are comparable to the structures we will generate later (such as point mutants).

For more information, see the RosettaCommons page describing how to prepare structures for Rosetta: https://www.rosettacommons.org/docs/latest/rosetta_basics/preparation/preparing-structures






\* In instances where you don't have a corresponding crystal structure, you'll need to approximate or predict one. See our section on 'structure prediction', coming soon.
### Evaluating the energy cost of a point mutation


### Energy cost of mutations on protein-ligand binding

### Energy cost of mutations on protein-protein binding

