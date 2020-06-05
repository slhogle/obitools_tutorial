# Getting started
Here is a quick intro to CSC Puhti. Generally, the CSC docs are good and most of what you will need is clearly documented. [Quick Start](https://docs.csc.fi/support/tutorials/puhti_quick/)

If you are unfamiliar with Linux/Unix based tools and the command line in generally I strongly suggest that you read through the [introduction to linux tutorials](https://docs.csc.fi/support/tutorials/env-guide/overview/) on the CSC Docs website.

# Log in to CSC Puhti and get the obitools singularity container
We're using a containerized version of the obitools for ease and simplicitiy. These are packaged using [`Singularity`](https://sylabs.io/) which is like Docker but better. There are good tutorials for using Singularity [here](https://sylabs.io/guides/3.5/user-guide/) and also [here](https://singularity-tutorial.github.io/).

Login
```{bash}
ssh username@puhti.csc.fi
```

Download Obitools image
```{bash}
singularity pull obitools.sif library://slhogle/base/obitools:latest
```

Check that it worked. You should see that the key was signed by Shane Hogle.
```{bash}
singularity exec obitools_latest.sif illuminapairedend
```
Should return: `No Entry to process` which is what we want. You can skip to the tutorial section at the bottom to put some real data through the pipeline and verify that things work. Generally, you just run the obitools commands the same as you would otherwise except you prefix each one with `singularity exec obitools.sif`.


# Running singularity containers on CSC Puhti
[General explanation for running with batch script](https://docs.csc.fi/computing/containers/run-existing/)

Obitools does not seem like it is multithreaded so you can run it just as well using an interactive shell on Puhti. [Here is an intro to interactive shell usage](https://docs.csc.fi/computing/running/interactive-usage/).

I would run like:
```{bash}
sinteractive --account project_2011234 --time 48:00:00 --mem 8000 --tmp 100
```
Then use obitool commands as usual. If you are want to run in interactive while you exit your remote session you can use sinteractive in a screen instance.

```
# take note of the login node you are currently on. 
# You will need the node number suffix (1 or 2) to reattach to your 
# specific screen session
screen -S myobitools

sinteractive --account project_2011234 --time 24:00:00 --mem 8000 --tmp 100

[run your obitool commands here]
```
Now you can exit from your screen session while obitools runs in the background. Press `Ctrl-a d` to detach from the screen.

You can even exit your ssh session on puhti. When you want to come back do where the login node is either 1 or 2 which you noted above:

```
ssh username@puhti-login[1|2].csc.fi
```
Now reattach your screen session:

```
screen -R myobitools
```

Now you see where you left off.

# Obitools Tutorial
This is really just to verify that I had this working correctly in a singularity container. The tutorial is taken from [here](https://pythonhosted.org/OBITools/wolves.html)

```{bash}
wget https://pythonhosted.org/OBITools/_downloads/wolf_tutorial.zip

unzip wolf_tutorial.zip

singularity exec obitools.sif illuminapairedend --score-min=40 \
-r wolf_tutorial/wolf_R.fastq wolf_tutorial/wolf_F.fastq > \
wolf_tutorial/wolf.fastq

singularity exec obitools.sif obigrep -p 'mode!="joined"' \
wolf_tutorial/wolf.fastq > wolf_tutorial/wolf.ali.fastq

singularity exec obitools.sif obihead --without-progress-bar \
-n 1 wolf_tutorial/wolf.ali.fastq

singularity exec obitools.sif ngsfilter -t wolf_tutorial/wolf_diet_ngsfilter.txt \
-u wolf_tutorial/unidentified.fastq wolf_tutorial/wolf.ali.fastq > \
wolf_tutorial/wolf.ali.assigned.fastq

singularity exec obitools.sif obiuniq -m sample \
wolf_tutorial/wolf.ali.assigned.fastq > wolf_tutorial/wolf.ali.assigned.uniq.fasta

singularity exec obitools.sif obiannotate -k count -k merged_sample \
wolf_tutorial/wolf.ali.assigned.uniq.fasta > $$ ; mv $$ wolf_tutorial/wolf.ali.assigned.uniq.fasta

singularity exec obitools.sif obistat -c count \
wolf_tutorial/wolf.ali.assigned.uniq.fasta | sort -nk1 | head -20

singularity exec obitools.sif obigrep -l 80 -p 'count>=10' \
wolf_tutorial/wolf.ali.assigned.uniq.fasta > \
wolf_tutorial/wolf.ali.assigned.uniq.c10.l80.fasta

singularity exec obitools.sif obiclean -s merged_sample -r 0.05 \
-H wolf_tutorial/wolf.ali.assigned.uniq.c10.l80.fasta > \
wolf_tutorial/wolf.ali.assigned.uniq.c10.l80.clean.fasta

singularity exec obitools.sif ecotag -d wolf_tutorial/embl_r117 \
-R wolf_tutorial/db_v05_r117.fasta wolf_tutorial/wolf.ali.assigned.uniq.c10.l80.clean.fasta > \
wolf_tutorial/wolf.ali.assigned.uniq.c10.l80.clean.tag.fasta

singularity exec obitools.sif obiannotate \
--delete-tag=scientific_name_by_db \
--delete-tag=obiclean_samplecount \
--delete-tag=obiclean_count \
--delete-tag=obiclean_singletoncount \
--delete-tag=obiclean_cluster \
--delete-tag=obiclean_internalcount \
--delete-tag=obiclean_head \
--delete-tag=taxid_by_db \
--delete-tag=obiclean_headcount \
--delete-tag=id_status \
--delete-tag=rank_by_db \
--delete-tag=order_name \
--delete-tag=order \
wolf_tutorial/wolf.ali.assigned.uniq.c10.l80.clean.tag.fasta > \
wolf_tutorial/wolf.ali.assigned.uniq.c10.l80.clean.tag.ann.fasta

singularity exec obitools.sif obisort -k count \
-r wolf_tutorial/wolf.ali.assigned.uniq.c10.l80.clean.tag.ann.fasta > \
wolf_tutorial/wolf.ali.assigned.uniq.c10.l80.clean.tag.ann.sort.fasta

singularity exec obitools.sif obitab \
-o wolf_tutorial/wolf.ali.assigned.uniq.c10.l80.clean.tag.ann.sort.fasta > \
wolf_tutorial/wolf.ali.assigned.uniq.c10.l80.clean.tag.ann.sort.ta
```
