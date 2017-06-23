
## Creating a parameter file
Parameter files provide rosetta with the bond descriptions (dihedral, angle, length) necessary to generate molecular groups. The conventional amino acids and several other molecular structures common among proteins already exist in rosetta's database. If you want to run rosetta with alternative molecules, you'll need to provide a parameter file for that molecular group.

Say that I want to model ligand binding to some enzyme that contains a prosthetic group. Take the structure for that enzyme, or the nearest homologous structure, and open it in PyMol. Align the structure to the pdb you're using for Rosetta modeling, making sure that the prosthetic group is in place. Select the noncanonical group at the active site, and click "File>Save Molecule" in the command bar. Pick "(sele)" from the resultant list and then click "Save global states." Make sure to change the filetype to ".sdf".

Once you've obtained the .sdf file, you'll likely need to open it in a software such as [BIOVIA discovery studio](http://accelrys.com/products/collaborative-science/biovia-discovery-studio/visualization-download.php). This software allows us to complete the hydrogen valencies in the molecule that are usually missing in a pdb files. Just click "Chemistry>Hydrogens>Add". The full version of thise program should be able to generate alternative conformations of the ligand as well. Make sure to save the file as '.sdf' after making your refinements.

Transfer your file to orchestra with ```scp ./file.sdf UserName@orchestra.med.harvard.edu:desiredDirectory/```

Then use "molfile_to_params.py" to convert the .sdf file to the .params format used by Rosetta. Specify a 3-letter code to refer to the molecular group by. You'll have to use this later in several places in order to write up the relationship between the molecule and protein.
~~~~
$ /programs/x86_64-linux/rosetta/3.8/main/source/scripts/python/public/molfile_to_params.py -n 3ID -p 3ID yourFile.sdf
~~~~

For regular ligands, the parameter file is ready to go. If the molecule is covalently attached to your protein, you'll need to add at least 2 lines describing that bond:
~~~~

~~~~

## Setting constraints
In order to perform a particular catalytic activity, enzymes must form a particular geometry of chemical groups at the active site. A constraints file can be used to specify this geometry along with an acceptable range of deviation, so that Rosetta penalizes structure that don't from the active configuration.

## Ligand binding
Given a small molecule and protein pair, we may want to elucidate the likely conformation of the ligand-bound complex. This is a process called ligand docking, wherein many positions and rotations of the small molecule are attempted and assessed.

In many enzymes, the ligand interacts with some cofactor within the protein. In this case, we must include additional information to specify the location and structure of the cofactor. If we have a crystal structure that includes the cofactor, we open the structure in PyMol and select the cofactor in order to save it as a '.sdf'. Next we use a python script in order to write a parameter file from the sdf.
~~~~
$ python /programs/x86_64-linux/rosetta/3.7/main/source/scripts/python/public/molfile_to_params.py cofactor.sdf
~~~~


