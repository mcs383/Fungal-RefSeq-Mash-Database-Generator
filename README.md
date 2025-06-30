# Fungal RefSeq Mash Database Generator
## An automated GitHub Actions workflow for creating up-to-date fungal mash sketch databases

This repository contains a GitHub Actions workflow designed to automatically build a comprehensive and up-to-date Mash sketch database from all available fungal reference genomes in the NCBI RefSeq database. This project was created to provide a fully automated, cloud-based method to generate a fresh Mash sketch database for the entire Fungi kingdom on demand, without requiring massive local computation or storage resources.

The workflow was adapted from [erinyoung/update_mash_dist](https://github.com/erinyoung/update_mash_dist) to support a yeast genomics project as a reusable resource for the research community.

## How it Works

This repository uses a single workflow (`refseq_fungal_mash_db_builder.yaml`) that automates the entire process using GitHub's own servers. The workflow consists of three main jobs that run in sequence:

1.  **Job 1: `list_genomes`**
    * Connects to the NCBI database and downloads a complete list of accession numbers for all currently available fungal reference genomes (NCBI `taxon 4751`).
    * Splits this master list into 10 smaller "chunk" files to enable parallel processing.
    * Uploads these chunk files as an artifact for the next job.

2.  **Job 2: `create_sketches`**
    * This is a parallel matrix job that runs 10 jobs simultaneously.
    * Each job is assigned one of the "chunk" files.
    * It processes its list one genome at a time in a disk-efficient loop: it downloads a single genome, creates a Mash sketch, merges it into a combined sketch for the chunk, and then deletes the downloaded files before starting the next one. This is crucial for running within the disk space limits of the GitHub servers.
    * Each job uploads its completed partial sketch as an artifact.

3.  **Job 3: `combine_sketches`**
    * This final job runs only after all 10 sketching jobs are complete.
    * It downloads all 10 of the partial sketch artifacts.
    * It uses `mash paste` to merge them into one final, comprehensive database sketch file.
    * It then compresses the final sketch and creates a formal **GitHub Release**, attaching the file for easy download.

## How to Use This Workflow

1.  **Fork this Repository:** Create your own copy of this repository by clicking the "Fork" button.
2.  **Enable Actions:** Go to the "Actions" tab of your newly forked repository. You may need to click a button to enable workflows.
3.  **Run the Workflow:**
    * In the Actions tab, select **"Build a mash sketch database of all fungal genomes in RefSeq"** from the list of workflows on the left.
    * Click the **"Run workflow"** button on the right.
    * You will be prompted to enter a **version tag** of the current version of the RefSeq database (e.g. `v230`, `v231`, etc.). This tag will be used to label your final release.
    * Click the final green **"Run workflow"** button.
4.  **Wait:** The process will take a few hours to complete. You can close your browser and computer; the job will continue running on GitHub's servers. You can monitor its progress on the "Actions" tab at any time.
5.  **Download Your Database:** When the workflow is complete (it will have a green checkmark ✅), go to the main page of your repository. On the right-hand side, click on **"Releases"**. Your new database will be there, attached as a `.msh.gz` file, ready to download.

## Once downloaded ensure the numebr of sketches matches the number of genomes in the current RefSeq Database

***Verifying the Database Contents***

To ensure your downloaded or newly-built Mash sketch file is complete, you can compare the number of genomes it contains with the current number of reference genomes available on NCBI.

***Step 1: Get the Current Genome Count from NCBI***

First, find out how many fungal reference genomes NCBI currently has in its database. This command queries the database directly. (Note: you must have the `ncbi-datasets-cli` tool installed and in your `PATH`).

```bash
datasets summary genome taxon 4751 --reference | grep "total_count"
This should print a line that looks something like this. Take note of the number.

  "total_count": 5088
```
  
***Step 2: Count the Genomes in Your Sketch File***

Next, use the mash info command to count how many individual genomes are inside your .msh file. Make sure you are in the directory containing your sketch file.

```bash
# Replace Your_Sketch_File.msh with your actual file name
mash info -t Your_Sketch_File.msh | wc -l
This will print a single number, which is the total count of sketches in your file.
```

***Step 3: Compare the Numbers***

The number from Step 2 should be identical or very close to the total_count from Step 1. If they match, your database is verified and ready to use!


## How to Cite

If you use this workflow or the database it generates in your research, please cite this GitHub repository. This helps us track the impact of our work and justify its continued maintenance.

You can use the following format:

> [Smardz, M. C.], & Young, E. (2025). *Fungal RefSeq Mash Database Generator: An automated GitHub Actions workflow for creating up-to-date fungal mash sketch databases* (Version 1.0). GitHub. [[(https://github.com/mcs383/Fungal-RefSeq-Mash-Database-Generator)](https://github.com/mcs383/Fungal-RefSeq-Mash-Database-Generator)]

## Acknowledgments

This workflow is a heavily modified version adapted specifically for fungi. The original concept and the robust parallel architecture were derived from the excellent work done by Erin Young for creating a bacterial database.
* **Original Repository:** [erinyoung/update_mash_dist](https://github.com/erinyoung/update_mash_dist)

Please creedit this repository if you use it for any published work. Thank you!

## License

This project is licensed under the **GNU General Public License v3.0**, in accordance with the original repository from which it was derived.
