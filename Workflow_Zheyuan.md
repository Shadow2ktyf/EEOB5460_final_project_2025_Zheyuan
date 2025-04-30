# EEOB5460 Final Project – Zheyuan's Workflow

## Project Directory Setup

1. Create a working project folder and navigate into it:

   ```bash
   mkdir EEOB5460_final_project_2025_Zheyuan
   cd EEOB5460_final_project_2025_Zheyuan
   ```

2. Create a folder to store downloaded FASTA sequences:

   ```bash
   mkdir fasta_files
   ```

## Download Reference Genomes (FASTA)

Use `curl` to retrieve publicly available coronavirus genomes from NCBI:

```bash
# WHU_1 (SARS-CoV-2 reference genome)
curl -L -o fasta_files/WHU_1.fasta "https://eutils.ncbi.nlm.nih.gov/entrez/eutils/efetch.fcgi?db=nuccore&id=MN908947.3&rettype=fasta&retmode=text"

# SARS-CoV (2003)
curl -L -o fasta_files/SARS-CoV.fasta "https://eutils.ncbi.nlm.nih.gov/entrez/eutils/efetch.fcgi?db=nuccore&id=AY278741.1&rettype=fasta&retmode=text"

# Bat SARS-like CoV (ZC45 strain)
curl -L -o fasta_files/Bat-SL-CoVZC45.fasta "https://eutils.ncbi.nlm.nih.gov/entrez/eutils/efetch.fcgi?db=nuccore&id=MG772933.1&rettype=fasta&retmode=text"

# MERS-CoV (EMC/2012)
curl -L -o fasta_files/MERS-CoV.fasta "https://eutils.ncbi.nlm.nih.gov/entrez/eutils/efetch.fcgi?db=nuccore&id=JX869059.2&rettype=fasta&retmode=text"
```

## Multiple Sequence Alignment (MSA) with MAFFT

1. Load the MAFFT module:

   ```bash
   module load mafft
   ```

2. Concatenate all downloaded FASTA files:

   ```bash
   cat fasta_files/*.fasta > all_sequences.fasta
   ```

3. Run MAFFT to perform the multiple sequence alignment:

   ```bash
   mafft --auto all_sequences.fasta > all_sequences_aligned.fasta
   ```

## Phylogenetic Tree Construction with IQ-TREE

1. Load the IQ-TREE module:

   ```bash
   module load iqtree2
   ```

2. Run IQ-TREE with automatic model selection and support value estimation:

   ```bash
   iqtree2 -s all_sequences_aligned.fasta -m TEST -bb 1000 -alrt 1000
   ```

   - `-s`: aligned input FASTA file
   - `-m TEST`: automatically test the best substitution model
   - `-bb 1000`: ultrafast bootstrap (1000 replicates)
   - `-alrt 1000`: SH-aLRT branch support test (1000 replicates)

3. IQ-TREE will generate the following output files:
   - `all_sequences_aligned.fasta.treefile`: main Newick tree
   - `all_sequences_aligned.fasta.iqtree`: model summary and log
   - Other auxiliary files: `.contree`, `.splits.nex`, `.model.gz`, `.bionj`, etc.

## Organize Output Files

Move all tree-related files to a dedicated folder:

```bash
mkdir tree_files
mv all_sequences_aligned.fasta.treefile all_sequences_aligned.fasta.iqtree all_sequences_aligned.fasta.contree all_sequences_aligned.fasta.mldist all_sequences_aligned.fasta.splits.nex all_sequences_aligned.fasta.model.gz all_sequences_aligned.fasta.bionj all_sequences_aligned.fasta.ckp.gz tree_files/
```

## Visualize the Phylogenetic Tree

Use the Interactive Tree Of Life (iTOL) platform:

1. Visit https://itol.embl.de
2. Upload `tree_files/all_sequences_aligned.fasta.treefile`
3. Customize branch styles, colors, and bootstrap labels
4. Export final figures as `.png`, `.pdf`, or `.svg`
