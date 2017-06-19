
## Setting constraints
In order to perform a particular catalytic activity, enzymes must form a particular geometry of chemical groups at the active site. A constraints file can be used to specify this geometry along with an acceptable range of deviation, so that Rosetta penalizes structure that don't from the active configuration.

## Ligand binding
Given a small molecule and protein pair, we may want to elucidate the likely conformation of the ligand-bound complex. This is a process called ligand docking, wherein many positions and rotations of the small molecule are attempted and assessed.

In many enzymes, the ligand interacts with some cofactor within the protein. In this case, we must include additional information to specify the location and structure of the cofactor. If we have a crystal structure that includes the cofactor, we open the structure in PyMol and select the cofactor in order to save it as a '.sdf'. Next we use a python script in order to write a parameter file from the sdf.
~~~~
$ python /programs/x86_64-linux/rosetta/3.7/main/source/scripts/python/public/molfile_to_params.py cofactor.sdf
~~~~


