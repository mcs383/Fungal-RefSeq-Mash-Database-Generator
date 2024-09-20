# Creating a MASH Reference

The mash reference that can be downloaded from (the mash documentaion)[https://mash.readthedocs.io/en/latest/data.html] is for RefSeq version 70.

I do not inherently have a problem with RefSeq version 70, but RefSeq is well past version 200 now. 

RefSeq updates four times year, and I needed an easy way to create and distribute a workflow mash sketch file.

To replicate the methods:
## Step 1. Download Datasets and Dataformat
```bash
wget https://ftp.ncbi.nlm.nih.gov/pub/datasets/command-line/v2/linux-amd64/datasets
wget https://ftp.ncbi.nlm.nih.gov/pub/datasets/command-line/v2/linux-amd64/dataformat
chmod +x datasets dataformat
```

## Step 2. Download Mash
```bash
wget https://github.com/marbl/Mash/releases/download/v2.3/mash-Linux64-v2.3.tar
tar -xvf mash-Linux64-v2.3.tar
```

## Step 3. Get a list of all the genomes

Note: this also changes how some of the names are represented

```bash
datasets summary genome taxon bacteria --reference --as-json-lines | \
  dataformat tsv genome --fields accession,organism-name --elide-header | \
  sed 's/\[//g' | \
  sed 's/\]//g' | \
  sed 's/["'\'']//g' | \
  sed 's/endosymbiont of /endosymbiont_of_/g' > \
  ids.txt
```

## Step 4. Download the reference files and sketch them

Note: Since this is done in Github Actions (GA), I need to keep everything below 30G. The best way to do this is to download the process each reference file individually, and then combine it to the whole. This obviously does not need to be followed if not under those same limitations.

```bash
while read line
do
  id=$(echo $line | awk '{print $1}')
  ge=$(echo $line | awk '{print $2}')
  if [ ! -n "$ge" ] ; then ge="unknown" ; fi
  sp=$(echo $line | awk '{print $3}')
  if [ ! -n "$sp" ] ; then sp="unknown" ; fi

  datasets download genome accession $id
  unzip ncbi_dataset.zip
  cp ncbi_dataset/data/*/*_genomic.fna ${ge}_${sp}_${id}.fasta
  if [ ! -f RefSeqSketches_${version}.msh ]
  then
    mash sketch ${ge}_${sp}_${id}.fasta -o RefSeqSketches_${version}
  else          
    mash sketch ${ge}_${sp}_${id}.fasta -o ${ge}_${sp}_${id}
    mv RefSeqSketches_${version}.msh tmp.msh
    mash paste RefSeqSketches_${version} tmp.msh ${ge}_${sp}_${id}.msh
    rm tmp.msh ${ge}_${sp}_${id}.msh
  fi

  rm ${ge}_${sp}_${id}.fasta
  rm -rf ncbi_dataset/
  rm ncbi_dataset.zip
  rm README.md
  rm md5sum.txt
done < ids.txt
```

## Step 5. Compress the sketch file
```bash
gzip RefSeqSketches_${version}.msh
```

## Step 6. Use

I did not try to do anything TOO fancy.
      
          

