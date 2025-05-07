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
curl -L -o fasta_files/WHU01.fasta "https://eutils.ncbi.nlm.nih.gov/entrez/eutils/efetch.fcgi?db=nuccore&id=MN988668.1&rettype=fasta&retmode=text"

curl -L -o fasta_files/WHU02.fasta "https://eutils.ncbi.nlm.nih.gov/entrez/eutils/efetch.fcgi?db=nuccore&id=MN988669.1&rettype=fasta&retmode=text"

curl -L -o fasta_files/CoV-2c-EM2012.fasta "https://eutils.ncbi.nlm.nih.gov/entrez/eutils/efetch.fcgi?db=nuccore&id=JX869059.2&rettype=fasta&retmode=text"

curl -L -o fasta_files/Bat-SL-CoVZC45.fasta "https://eutils.ncbi.nlm.nih.gov/entrez/eutils/efetch.fcgi?db=nuccore&id=MG772933.1&rettype=fasta&retmode=text"

curl -L -o fasta_files/SARS-CoV.fasta "https://eutils.ncbi.nlm.nih.gov/entrez/eutils/efetch.fcgi?db=nuccore&id=AY278741.1&rettype=fasta&retmode=text"
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

Turn your environment to Jupyter Notebook:

```python
from Bio import Phylo
import matplotlib.pyplot as plt

label_map = {
    "AY278741.1": "SARS-CoV",
    "MG772933.1": "Bat-SL-CoVZC45",
    "JX869059.2": " CoV-2c-EM2012",
    "MN988668.1": "WUH01",
    "MN988669.1": "WUH02"
}

tree = Phylo.read("all_sequences_aligned.fasta.treefile", "newick")

for clade in tree.find_clades():
    if clade.name in label_map:
        clade.name = label_map[clade.name]

fig = plt.figure(figsize=(8, 6))
Phylo.draw(tree, do_show=False)
plt.savefig("tree_labeled.png", dpi=300)
```

