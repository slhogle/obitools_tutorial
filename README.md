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
