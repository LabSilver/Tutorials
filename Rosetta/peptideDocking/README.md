The tutorial below explains how to use the FlexPepDocking application for sampling the docking configurations of a short (5-15aa) peptide on a local area of a binding protein. Fragment picking is also described, as it's necessary for flexible sampling of the peptide should its structure be unknown.

## Fragment picking
Rosetta has various ways to perturb bond angles in existing backbones and side chains, but sometimes structures must be built piece by piece without the benefit of an starting fold. In these cases, structures are built by small 3, 5, and 9 amino acid *fragments*. Fragments are potential bond angles found in existing crystal structures that correspond to short strings of amino acids within the input sequence (provided as a '.fasta' file). We can produce fragments for a target protein with the 'fragment_picker' application:

We will need: An extended chain '.pdb' for the desired protein sequence, a '.fasta' file for that sequence, a secondary structure prediction for that sequence, a database of possible fragments, and a weights value for deciding which fragments are good. We'll also want to provide flags as instructions to the picker.

**Creating an extended chain .pdb file,** To obtain an extended peptide with the desired sequence, we can use the 'build' feature in pymol. Just enter the following command but substitute in your amino acid sequence:
~~~~
for aa in "DCAHWLGELVWCT": cmd._alt(string.lower(aa))
~~~~

Use 'File->Save Molecule...' in order to obtain the extended amino acid as a .pdb file.

**Creating a '.fasta' file corresponding to the peptide.** We can generate a corresponding '.fasta' by using the perl script 'getFastaFromCoords.pl':
~~~~
$ /programs/x86_64-linux/rosetta/3.8/tools/perl_tools/getFastaFromCoords.pl -pdbfile {yourPDB} -chain {yourChain}
~~~~

**Obtaining a secondary structure prediction.** The Bloomsbury Centre for Bioinformatics has a [convenient server](http://bioinf.cs.ucl.ac.uk/psipred/) for secondary structure prediction. Submit your protein sequence and once it has completed, go the the 'Downloads' tab and click 'PSIPRED raw scores in plain text format'.

Upload these files to Orchestra by opening a commandline and navigating to the directory contianing these files. Then enter:
~~~~
$ scp {myfile} {usrName}@orchestra.med.harvard.edu:{desiredDirectory}
~~~~
You'll also want to rename each of your files to a 5 character code, or the fragment picker will error. Also change the extension of your secondary structure file to '.psipred.ss2':
~~~~
$ mv {myfile.file} {XXXXX.file}
~~~~

**Databases, weights, and flags**
We want to pass our inputs to the fragment picker, along with instructions to the fragment picker and directions to the database and weights. Create a weights file with ```$ vi simple.wts``` and paste ('I', SHIFT+INS) in the following:
~~~~
# score name          priority  wght   max_allowed  extras
RamaScore               400     2.0     - psipred
SecondarySimilarity     350     1.0     - psipred
FragmentCrmsd             0     0.0     - psipred
~~~~
Next create your flags file with ```$ vi picker.flags``` and paste:
~~~~
# Input databases
-database                       /programs/x86_64-linux/rosetta/3.8/main/database/
-in:file:vall                   /programs/x86_64-linux/rosetta/3.8/tools/fragment_tools/vall.apr24.2008.extended
# Query-related input files
#-in::file::checkpoint          XXXXX.checkpoint
-in:file:fasta                  ./XXXXX.fasta
-in:file:s                      ./XXXXX.pdb
-frags:ss_pred                  ./XXXXX.psipred.ss2 psipred  # second input is the ID used by '.wts' file
# Weights file
-frags:scoring:config           ./simple.wts
# What should we do?
-frags:bounded_protocol
# three-mers only, please
-frags:frag_sizes               3 5 9
-frags:n_candidates             200
-frags:n_frags                  200
# Output
-out:file:frag_prefix           ./output/frags
-frags:describe_fragments       ./frags.fsc
~~~~

Create an output directory with ```mkdir ./output``` and then run the code with:
~~~~
$ /programs/x86_64-linux/rosetta/3.8/main/source/bin/fragment_picker.linuxgccrelease @picker.flags
~~~~
If your protein is large, you may want to submit this with bsub.

In the output directory, you should obtain three files: 'frags.200.3mers', 'frags.200.5mers', and 'frags.200.9mers'. These can be fed into Rosetta applications that generate *ab initio* chain conformations for the particular input sequence you provided.



## Peptide binding
Various proteins can bind short peptide chains, which are often flexible. Even in short peptides there are many degrees of freedom, so it can be important to consider the numerous conformations of the peptide when searching for the lowest energy binding mode. The Rosetta application, FlexPepDocking does just this.

**Output: what we can find**
Primary docking conformations, which will appear as the lowest energy conformations of unique binding modes (as grouped by RMSD).

**Input: what we need to start**
PDB file for the peptide-binding protein. See section on [Preparing Protein Structures](https://github.com/LabSilver/Tutorials/tree/master/Rosetta#preparing-protein-structures).
PDB file the peptide. See section on [Fragment Picking](https://github.com/LabSilver/Tutorials/blob/master/Rosetta/peptideDocking/README.md#fragment-picking) for instructions on building peptides in pymol.
Fragment files for the peptide. See section on Fragment Picking.
Flags, see below:
~~~~
#inputs:
-s COMPLEX_ppk.pdb
-frag3 frags/frags.200.3mers
-frag9 frags/frags.200.9mers
-flexPepDocking:frag5 frags/frags.200.5mers
-nstruct 5
-database /programs/x86_64-linux/rosetta/3.8/main/database/
#outputs:
-out:pdb_gz
-out:file:silent_struct_type binary
-out:file:silent decoys.silent
-scorefile score.sc
#fragment picker flags:
-flexPepDocking:frag5_weight 0.25
-flexPepDocking:frag9_weight 0.1
#flexpepdock flags:
-flexPepDocking:lowres_abinitio
-flexPepDocking:pep_refine
-flexPepDocking:flexpep_score_only
#packing flags:
-ex1
-ex2aro
-use_input_sc
#mute logging:
-mute protocols.moves.RigidBodyMover
-mute core.chemical
-mute core.scoring.etable
-mute protocols.evalution
-mute core.pack.rotamer_trials
-mute protocols.abinitio.FragmentMover
-mute core.fragment
-mute protocols.jd2.PDBJobInputter
~~~~

Before running docking, prepack the peptide and protein (optimize their side-chains when separate from one another) using these prepack flags:
~~~~
-s COMPLEX.pdb
-ex1
-ex2aro
-use_input_sc
-flexpep_prepack
-nstruct 1
~~~~
Execute the following, and move the resultant output file 'COMPLEX_0001.pdb' to 'COMPLEX_ppk.pdb':
~~~~
$ /programs/x86_64-linux/rosetta/3.8/main/source/bin/FlexPepDocking.linuxgccrelease @prepack.flags
~~~~
Docking can be performed on 'COMPLEX_ppk.pdb' by:
~~~~
$ /programs/x86_64-linux/rosetta/3.8/main/source/bin/FlexPepDocking.linuxgccrelease @pepdocking.flags
~~~~
The outputs are stored in 'decoys.silent' as 'COMPLEX_ppk_0001', 'COMPLEX_ppk_0001'... retrieve the corresponding .pdb files with 'extractpdbs'.
