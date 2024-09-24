# Project 1: MD Simulations of RNA Acridine:G-quadruplexes

Note: You are strongly encouraged to work together.  However, each person is responsible for producing and submitting their own work.

Overview: We will perform CHARMM MD simulations with NAMD of a RNA G-quadruplex in explicit solvent on the DEAC cluster.  Using the production run of the simulation, we will use VMD and other programs to visualize trajectories and perform analyses on them.  

## Part 2: Setting up and Running a NAMD MD simulation on the DEAC HPC Cluster

Website References: [ NAMD Tutorial ](http://www.ks.uiuc.edu/Training/Tutorials/namd/namd-tutorial-unix-html/index.html)

NAMD is a leading empirical force field MD simulation software suite that can use the CHARMM force field to perform MD simulations of molecules. In this section, you will use the DEAC cluster to perform MD simulations of a G-quadruplex. 

To date, no one has ever performed these MD simulations so you are the first. I highly encourage you to work together for technical assistance and ask me for help too. No two assignments will be the same, and I have no idea what the results will be. Isn’t that exciting?

Note: This portion of the project will require that you have already mastered the topics covered in Project 0 and Project 1, Part 1. If this is not the case, be sure to reference those documents.

1. Log into the DEAC HPC Cluster (see Project 0, Part 2)
2. Change directory to your project directory (see Part 0, Part 1):
   ```
   cd /deac/phy/classes/phy620/[username]
   ```
   where [username] should be replaced by your username.

3. Clone this repository by using the following command (See Part 0, Part 4):
  ```
  git clone [repository url]
  ```

  This repository has a subdirectory called __g-quad__ and you will perform MD simulations of a G-quadruplex there. In the direcotry are three subdirectories: 1) input_data, 2) setup, and 3) simulations. The following files have been set up for you to run your MD simulations:

  | input_data/45S.pdb | PDB file containing the coordinates of a G-quadruplex |
  | input_data/k.pdb | PDB file containing the coordinates of potassium ions in the G-quadruplex |
  | setup/top_all36_prot_na.rtf | CHARMM force field topology file |
  | setup/setup.pgn | VMD script to convert the 45S.pdb and k.pdb file into a format that can be readily used by NAMD. |
  | setup/solvate.pgn | VMD script to add a water box and NaCl ions. |
  | setup/Klean.sh | A “reset” script. If you get in trouble and you need to start over, run this script. It should get you back to the beginning (I think, I hope, knock on wood). |
  | simulations/par_all36_prot_na.prm | CHARMM force field parameter file |
  | simulations/equil_1e.conf | NAMD configuration file for running the “equilibration” portion of the MD simulation. |
  | simulations/dyna.temp | NAMD configuration file template for running the “production run” portion of the MD simulation. |
  | simulations/gen_scripts.sh | Script to generate NAMD configuration files during production run. |
  | simulations/md.slurm | SLURM script for running MD simulation on the cluster. |

4. First we will make sure the PDB files are okay. We wil use VMD to visualize 45S.pdb and k.pdb file.

On the DEAC cluster, programs are made available to you only if you request it. To load the VMD program, you must load the module using the following command:
```
module load apps/vmd/1.9.4a57
```
Then type “vmd” and the program will load up as before. Open up and load 45S.pdb by going to the “VMD Main” window and selecting File --> New Molecule… . In the “Molecule File Browser” window, click on the “Browse…” button, locate the 45S.pdb file, and double-click on it. Then click on the “Load” button. This will load up the G-quadruplex image.

![image](https://github.com/user-attachments/assets/bd56b021-1612-44f2-a439-43d2b26cf6d4)
![image](https://github.com/user-attachments/assets/38590363-65d3-49cc-8bdf-917e14945dd4)

In the “Load files for:” dropdown list, select “New Molecule”. Click on the “Browse…” button locate the k.pdb file and double-click on it. Then click on the “Load” button. This will load up the potassium ions image. You will not be able to see it (because it’s just two potassium atoms), but the image will be zoomed on them.

In the “VMD Main” window, select Graphics --> Representations… . A “Graphical Representation” window will open.

If you make sure that the “Selected Molecule” is k.pdb and change the “Drawing Method” to VMD, you will now be able to see the potassium ions. You may wish to zoom out a bit though. The resulting image will look something like this:

![image](https://github.com/user-attachments/assets/da819f6a-eac0-4953-ab4b-1bdbbc6b4589)
![image](https://github.com/user-attachments/assets/f6706571-9aad-4dcc-a8c5-85cf0b167a18)
![image](https://github.com/user-attachments/assets/d5be960e-fcbe-4fc5-aaee-cafeee7ff8cd)

If you get the above image (or something very similar to it), you can be satisfied that your files are okay. Exit the VMD program.

5. To look at what a PDB file really looks like, we will use a text editor. We are going to use __nano__.

Notice that the PDB file has several columns of information. The 1st column (“ATOM”) denotes that the line contains information about an atom, 2nd column is the atom number, the 3rd column is the atom type, 4rd column is the nucleotide type (or residue for proteins), 5th column is the chain id, 6th is the nucleotide number (or residue for proteins), 7th, 8th, and 9th columns are the x-, y-, and z- coordinates, 10th is not important anymore and is always 1.00, and 11th is the thermal factors (how much the atom fluctuates according to experiments).

Use the scroll bar on the far right to scroll to the bottom of the file. Save the number of nucleotides in the PDB file. This is different for each of you because it depends on which PDB file you received.

Copy the 45S.pdb and k.pdb files to the “setup” subdirectory.

6. Next we will use the VMD script setup.pgn to convert the 45S.pdb and k.pdb file into a set of NAMD-style files that it is used to handling. 

This is a generic script for any biomolecule. You will need to modify it using the __nano__ program. 
  a. Open “setup.pgn” using __nano__.
  
  b. Replace all instances of “\[ID\]” (including brackets!) with “45S”.

Run the VMD script using the following command:
```
module load apps/vmd/1.9.4a57
vmd -dispdev text -e setup.pgn
```

A lot of text will show up on the screen. Unless “ERROR” or “MOLECULE MISSING!” or other ominous sounding text appears, you’re fine. “Warning” is not sufficiently worrisome.

To verify that everything really is fine, type “vmd 45S.namd.psf 45S.namd.pdb”. An image very similar to the one from step 4 will appear.

7. Next we will use the VMD script solvate.pgn file to add a water box and sodium and chloride ions to our G-quadruplex. Again, this is a generic script for any biomolecule. You will need to modify it using the __nano__ program. 

  a. Open “solvate.pgn” using __nano__. 
  
  b. Replace all instances of “[ID]” with “45S”.

After you’re done, save your changes and exit out of the program

Run the VMD script using the following command:
```
vmd -dispdev text -e solvate.pgn
```

To verify that everything really is fine, type “vmd 45S_wbi.psf 45S_wbi.pdb”. 

![image](https://github.com/user-attachments/assets/c181cbe4-1c81-4ebe-95f3-9ef1f2b9f6a7)

My screen looks differently from yours only because I selected the ions and changed their drawing method as VDW. Exit VMD.

8. Use __nano__ to open up the 45S_wbi.pdb file. On the first line, first column is “CRYST1”. The next three sets of numbers are the x-, y-, and z-dimensions of the water box. Save the water box dimensions. Exit out of the program.

9. Next we will use the NAMD configuration file equil_1e.conf file to read in the G-quadruplex + water box + ions and run 50 ps equilibration by raising the temperature from 0 K to 298 K. We will be using periodic boundary conditions and various cutoff methods we discussed in class. Again, this is a generic script for any biomolecule. You will need to modify it using the __nano__ program. 

  a. Open “equil_1e.conf” using __nano__. 
  
  b. Replace all instances of “[ID]” with “45S”.
  
  c. Replace the instances of [XPBC], [YPBC], and [ZPBC] with the x-, y-, and z-dimensions of the water box (to the tenth place precision) from Step 8.

10. Next we will use the NAMD configuration file template file dyna.temp to read in the MD simulation state from the previous step and continue for 1 ns. Again, this is a generic script for any biomolecule. You will need to modify it using the VS Code program.

  a. Open “dyna.temp” using __nano__. 
  
  b. Replace all instances of “[ID]” with “45S”.

After you’re done, save your changes and exit out of the program

11. Next we will set up 10 ns of MD simulations of 1 ns parts using the dyna.temp file. Type the following command:

```
for ((i=2; i<=11; i++)); do ./gen_scripts.sh $i; done
```

This is a nifty little program that I wrote that will write NAMD configuration file for doing the 1st nanosecond of MD simulation and then the 2nd nanosecond and so forth until the end.

12. Next we will set up the SLURM script for running MD simulations on the cluster. Again, this is a generic script for any biomolecule. You will need to modify it using the VS Code program..

  a. Open “md.slurm” using __nano__. 
  
  b. Replace all instances of “[username]” with your username and [ID] with “45S”.

After you’re done, save your changes and exit out of the program.

13. In steps 9-12, we set up the equilibration and production run portions of the MD simulation and even the SLURM script to request computers from the DEAC cluster to run it, but we did not actually run it. (No, we have not been using the full power of the DEAC cluster yet.) We will be using the SLURM queuing system.

How to use the SLURM queuing system:
Three main commands:
  1.	sbatch -> submit to the queue.
  2.	squeue -> check the status of the queue.
  3.	scancel -> cancel a submission to the queue.

To submit your md.slurm script to the DEAC cluster, type “sbatch md.slurm”. The check on whether everything is okay, type “squeue –u [username]” where [username] is your username. If it is running, your job will show up as below. 

![image](https://github.com/user-attachments/assets/cc67b75d-21c1-43d3-b1f8-3554b7d94063)

In the “ST” column, an “R” indicates that it is currently running and a “PD” indicates that it is waiting (or pending) in the queue.

I strongly recommend that you periodically “baby-sit” your MD simulation until it is finished.  Note: This should take at least 24 hours, but no longer than two days. Wait patiently until your simulations are finished. If something seems wrong, contact me.

14. When your MD simulation is finished, you can visualize it using VMD and even save it as a video.

__Turn in the movie on Canvas.  Feel free to use any artistic license you please.__
