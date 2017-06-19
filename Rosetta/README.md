# Rosetta tutorials

## Getting started
<details><summary> Command line, logging in, sourcing BioGrids</summary><p>
These guides are to be executed in Orchestra with the BioGrids environment. Obtain access to the HMS computing cluster, Orchestra at https://rc.hms.harvard.edu/. You can use Orchestra to perform intensive computational tasks, just log in with a Unix-based command line (the default terminal for Macs; or Cygwin for PCs, free at https://cygwin.com/install.html). The command to log in is:

``$ ssh -y usrNM@orchestra.med.harvard.edu``

Note: I use the dollar sign ($) to indicate entries meant for the command line, otherwise I'm referring to script editing with vim. Don't actually type the dollar sign.

Initialize BioGrids on your Orchestra account by sourcing the Biogrids environment:
``$ source /programs/biogrids.shrc``
You'll only need to do this once, and then it'll automatically initialize when you log in.
 
More information on BioGrids setup can be found here: https://wiki.med.harvard.edu/Orchestra/BioGrids
Please contact help@biogrids.org if you run into any problems.

Additionally, before we get started, your life will be made much simpler if you do the following:
```
$ echo "alias cleanpdb='python /programs/x86_64-linux/rosetta/3.8/tools/protein_tools/scripts/clean_pdb.py'" >> ~/.bashrc
$ echo "alias testrelax='/programs/x86_64-linux/rosetta/3.8/main/source/bin/relax.linuxgccrelease'" >> ~/.bashrc
$ echo "alias extractpdbs='/programs/x86_64-linux/rosetta/3.8/main/source/bin/extract_pdbs.linuxgccrelease -database /programs/x86_64-linux/rosetta/3.8/main/database'" >> ~/.bashrc
$ echo "alias shortrelax='bsub -q short -W 12:00 /programs/x86_64-linux/rosetta/3.8/main/source/bin/relax.linuxgccrelease'" >> ~/.bashrc
```
``exit`` and log back into Orchestra to implement these shortcut commands.
</p></details>

## Rosetta
Skip to a specific protocol if you want to:
* Predict a protein structure from scratch
* Predict a protein strucure from homology
* Predict orientations of a protein-protein complex
* Predict the binding mode for a small molecule
* Predict the binding mdoe for a peptide
* Screen mutations of a known protein for stability
* Screen mutations for protein-protein interactions
* Screen mutations for protein-peptide interactions
* Screen mutations for protein-small molecule binding

You can find any of the requisite files for these tutorials in my folders:
~~~~
$ cd /home/njr8/public/tutorials/
~~~~
(Navigating in commandline: 'cd' stands for change directory and allows us to move between folders. Beginning a file address with './' will begin the address from the folder you are currently in. Beginning with '../' will begin the address *one folder back* from the current folder. Beginning with '/' will start from the root directory. The other user accounts on orchestra can thus be accessed with '/home/{username}')

### Preparing protein structures
Before we can get started in full, we need to obtain a pdb structure of the protein in question.

Ideally, we can just clean the junk crystallization molecules off of a known structure\*:
~~~~
$ cleanpdb {your xtal} {chainIDs}
~~~~
\* In instances where you don't have a corresponding crystal structure, you'll need to approximate or predict one from scratch (*ab initio*) or from known homologous structures.

This script will only keep the essential atoms that are used by Rosetta, and convert non-canonical amino acids into the nearest canonical counterpart. The output will be named {xtal}\_{chainID}.pdb. After cleaning the pdb, it's important that we "relax" it, so that resulting thermodynamic scores are comparable to the structures we will generate later (such as point mutants).

First, we'll want to create a "flags" file that tells Rosetta what sorts of limitations we want to impose in the relaxation. In the commandline enter:
~~~~
$ vi example.flags
~~~~
Then pressing 'ctrl+I' to write to the new file, paste (middle mouse button, or shift+insert) the following into the file:
~~~~
-database /programs/x86_64-linux/rosetta/3.8/main/database    # give the directory of the rosetta databases
-relax:constrain_relax_to_start_coords                        # force relax to operate near the original coordinates
-relax:coord_constrain_sidechains                             # keep sidechains from moving
-relax:ramp_constraints false                                 # keep constant constraint weights
-ex1                # extra rotamer angles 1
-ex2                # extra rotamer angles 2
-use_input_sc       # begin with sidechain angles from a starting input structure
-flip_HNQ           # include alternate protonation states of H, N, and Q
-no_optH false      # allows optimization of hydrogens at the end of the run
~~~~
Further flag descriptions and other options can be found here:
https://www.rosettacommons.org/docs/latest/rosetta_basics/options/options-overview

We can relax the structure of uncleaved rat CBG (ID: 2v95, here we have cleaned chain A) by entering:
~~~~
$ bsub -q short -o output1 -W 12:00 /programs/x86_64-linux/rosetta/3.8/main/source/bin/relax.linuxgccrelease @example.flags -s 2v95_A.pdb
~~~~
or with our convenient alias:
~~~~
$ shortrelax @example.flags -s 2v95A.pdb
~~~~

Rosetta computations are intensive, so we need to reserve memory in Orchestra for however long we expect the code to run. We use ``bsub -q fast -o output1`` to submit to the "fast" queue and contain the junk spat out of Rosetta in output1. ``-W 12:00`` reserves 12 hour of computational time.

The output of this relax run will be a single structure '2v95A_0001.pdb', and 'score.sc' a series of different score metrics for that structure. The flags we provided above constrain the output to be approximately unchanged from the input structure, but in more exhaustive searches for the lowest energy structure we might want to do a lot more sampling. If we include a flag ``-n 1000`` we can create a random sampling of a thousand output structures. 

But generating a thousand .pdbs would massively clutter our workspace, so we concisely store those structures in a \"silent\" file by including the flag ``-out:silent``. In this case, you'd sort the score file using the following commandline in order to see the details for the best output (lowest score).
~~~~
$ sort -nk2 score.sc | head -1
~~~~
We can recover specific pdbs from the silent file by entering the IDs we want with the following command:
~~~~
$ extractpdbs -in:file:silent {silentFile} -in:file:tags {ID1} {ID2} {etc}
~~~~
And now we have the pdb corresponding to the best scoring relaxed structure!


#### Regarding LSF submissions
Later you may want to perform jobs that take longer than 12 hours, and may need to pick a different queue that supports these longer jobs. See these pages for information on which queue to submit your jobs to:
https://wiki.med.harvard.edu/Orchestra/IntroductionToLSF#Which_queue
https://wiki.med.harvard.edu/Orchestra/ChoosingAQueue

bsub doesn't recognize aliases, so we needed to use the full location of the relax function here. A way to simplify your life is to add an alias to your ``~\.bashrc`` that includes the bsub details but leaves the flags and structure for you to determine:
~~~~
$ echo alias "BSUB-rosetta-relax='bsub -q short -o output1 -W 12:00 /programs/x86_64-linux/rosetta/3.7/main/source/bin/relax.linuxgccrelease'" >> ~/.bashrc
~~~~
And you'll then be able to perform relaxations with the simple command: ``BSUB-rosetta-relax @{flags} -s {pdb}``

#### Further questions
For more information, see the RosettaCommons page describing how to prepare structures for Rosetta: https://www.rosettacommons.org/docs/latest/rosetta_basics/preparation/preparing-structures

### Structure prediction: homology reference


### Structure prediction: ab initio, fragment database

### Ligand docking
Given a small molecule and protein pair, we may want to elucidate the likely conformation of the ligand-bound complex. This is a process called ligand docking, wherein many positions and rotations of the small molecule are attempted and assessed.

In many enzymes, the ligand interacts with some cofactor within the protein. In this case, we must include additional information to specify the location and structure of the cofactor. If we have a crystal structure that includes the cofactor, we open the structure in PyMol and select the cofactor in order to save it as a '.sdf'. Next we use a python script in order to write a parameter file from the sdf.
~~~~
$ python /programs/x86_64-linux/rosetta/3.7/main/source/scripts/python/public/molfile_to_params.py cofactor.sdf
~~~~



### Evaluating the energy cost of a point mutation

### Energy cost of mutations on protein-ligand binding

### Energy cost of mutations on protein-protein binding

