
# EGFR Protein Structure Prediction and Molecular Docking Using AlphaFold2 and GNINA

This repository contains a complete beginner-friendly computational biology workflow for the **EGFR gene/protein**. The notebook follows the biological order of information flow:

**DNA → mRNA → protein → AlphaFold2 structure prediction → GNINA molecular docking**

The project starts from an EGFR DNA FASTA sequence, converts it into mRNA, translates the mRNA into a protein sequence, prepares the EGFR kinase domain for AlphaFold2 or ColabFold, and then performs docking of erlotinib into the EGFR binding pocket using GNINA.

<img width="800" height="300" alt="workflow_overview" src="https://github.com/user-attachments/assets/71f686dd-bc04-4967-85b2-e2d646ff48cb" />


---

## Project Aim

The aim of this project is to demonstrate a complete in silico drug-discovery style workflow using EGFR as the target protein.

The workflow explains how a DNA sequence can be converted into a protein sequence and how that protein sequence can be used for structure prediction. After structure-related preparation, the project performs molecular docking to study how a small-molecule inhibitor, **erlotinib**, may bind to the EGFR kinase domain.

This project is useful for students learning:

- The central dogma of molecular biology
- FASTA sequence handling
- DNA transcription
- mRNA translation
- Protein structure prediction using AlphaFold2 or ColabFold
- Protein-ligand docking using GNINA
- Docking score interpretation

---

## Biological Background

### EGFR

EGFR, or epidermal growth factor receptor, is a receptor tyrosine kinase. It is involved in cell growth, cell division, and signaling pathways. Mutations or overactivity of EGFR are associated with several cancers, especially some lung cancers.

EGFR has multiple regions, including an extracellular ligand-binding region, a transmembrane region, and an intracellular kinase domain. The kinase domain is important because many EGFR inhibitors bind in or near its ATP-binding pocket.

### Central Dogma: DNA to RNA to Protein

The first part of the notebook follows the central dogma of molecular biology:

1. DNA stores genetic information.
2. mRNA is produced from DNA during transcription.
3. Protein is produced from mRNA during translation.

<img width="800" height="300" alt="central_dogma" src="https://github.com/user-attachments/assets/ad61ba05-c1ef-43a9-ab47-af9d57e8a9e1" />


In this notebook, the DNA sequence is expected to be a coding sequence. That means it should represent the protein-coding portion of EGFR without introns. This is important because normal genomic DNA contains introns, which must be removed before translation. If a full genomic sequence with introns is used directly, the translated protein may be incorrect.

### AlphaFold2

AlphaFold2 predicts a protein's three-dimensional structure from its amino acid sequence. In this project, the EGFR kinase domain sequence is prepared for AlphaFold2 or ColabFold. The predicted PDB file is then uploaded back into the notebook and visualized.


### GNINA

GNINA is a molecular docking tool that predicts how a ligand may bind inside a receptor binding pocket. It can report traditional docking scores and CNN-based scores. In this notebook, erlotinib is docked into the EGFR kinase domain binding site.

<img width="800" height="300" alt="gnina_docking_concept" src="https://github.com/user-attachments/assets/920fe33f-739b-43a7-b5ed-929b105ecc7e" />


---

## Software and Tools Used

The notebook uses the following tools:

| Tool | Purpose |
|---|---|
| Google Colab | Cloud-based notebook environment |
| Python | Main programming language |
| Biopython | Reading FASTA files, transcription, translation, sequence writing |
| pandas | Storing docking scores in table/CSV format |
| NumPy | Calculating ligand coordinate center for docking box |
| py3Dmol | Visualizing protein and ligand structures in the notebook |
| Open Babel | Converting ligand file formats |
| AlphaFold2 / ColabFold | Predicting EGFR kinase-domain 3D structure |
| GNINA | Performing molecular docking |
| RCSB PDB structure 1M17 | EGFR kinase domain structure containing erlotinib ligand |

---

## Input Files

### Required input file

The first required input is an EGFR DNA FASTA file.

Recommended filename:

```text
egfr_dna.fasta
```

### AlphaFold2 PDB input

The notebook does not run the full AlphaFold2 model directly. Instead, it prepares the FASTA sequence for AlphaFold2 or ColabFold. After the structure is predicted externally, upload the predicted `.pdb` file into the notebook when requested.

---

## Step-by-Step Workflow

## Step 1. Check GPU Availability

The notebook begins by checking whether a GPU is available in Google Colab. AlphaFold2/ColabFold and molecular docking can be computationally demanding. A GPU helps speed up structure prediction and may improve performance for deep-learning-based scoring methods. The `nvidia-smi` command displays the available NVIDIA GPU, GPU memory, driver version, and current usage.

Code used:

```bash
!nvidia-smi
```

### Expected result

If a GPU is enabled, the output should show information such as the GPU name, memory, and CUDA version. If no GPU appears, enable it in Colab using:

```text
Runtime → Change runtime type → Hardware accelerator → GPU
```

---

## Step 2. Install Required Libraries and Tools

The notebook installs the required Python libraries and Linux command-line tools.

Main packages installed:

```python
biopython
pandas
py3Dmol
openbabel
wget
zip
```

Different parts of the workflow require different tools:

- **Biopython** reads and writes FASTA files and performs transcription and translation.
- **pandas** creates a table from GNINA docking scores.
- **py3Dmol** visualizes protein and ligand structures inside the notebook.
- **Open Babel** converts ligand molecular file formats.
- **wget** downloads the EGFR crystal structure from the Protein Data Bank.
- **zip** can be used to compress project files for submission or sharing.
---

## Step 3. Upload the EGFR DNA FASTA File

The project starts from DNA. In this step, the user uploads the EGFR DNA FASTA file. The DNA file is the starting point of the workflow. All later sequence files are generated from this DNA sequence. The notebook assumes the uploaded DNA is a coding sequence. If the input contains introns or non-coding regions, the translated protein sequence may not match the real EGFR protein.

Code used:

```python
from google.colab import files
uploaded = files.upload()
```
```text
DNA FASTA → mRNA FASTA → protein FASTA
```
---

## Step 4. Read the DNA Sequence

The notebook automatically detects FASTA files and reads the DNA sequence using Biopython.

It reads the first detected FASTA file and stores the sequence as `egfr_dna`.

The code also cleans the sequence by keeping only valid DNA bases:

```text
A, T, G, C
```

Any invalid letters, spaces, or formatting characters are removed.

Before transcription and translation, the sequence must be clean and biologically valid. Invalid characters can cause incorrect transcription, translation errors, or wrong protein output.

### Output:

The notebook writes a cleaned DNA FASTA file:

```text
egfr_dna.fasta
```

This file is later copied into:

```text
EGFR_Project/data/egfr_dna.fasta
```

---

## Step 5. Transcribe DNA into mRNA

Transcription converts DNA into messenger RNA.

During transcription:

- DNA thymine `T` becomes RNA uracil `U`.
- The coding information is copied into mRNA.
- The mRNA sequence contains codons that can be translated into amino acids.

Example:

```text
DNA:  ATG GAA TTT
mRNA: AUG GAA UUU
```

Biopython performs transcription using:

```python
egfr_mrna = egfr_dna.transcribe()
```

The resulting mRNA sequence is saved as:

```text
egfr_mrna.fasta
```
---

## Step 6. Translate mRNA into Protein

Translation converts the mRNA sequence into an amino-acid sequence.During translation, the ribosome reads mRNA in groups of three nucleotides called codons. Each codon corresponds to one amino acid.

AlphaFold2 does not use DNA or RNA as input. It predicts protein structure from an amino-acid sequence. Therefore, the translated protein is required before structure prediction can begin.

Example:

```text
mRNA codon: AUG
Amino acid: Methionine / M
```

Biopython translates the mRNA sequence using:

```python
egfr_protein = egfr_mrna.translate(to_stop=True)
```

The option `to_stop=True` means translation stops at the first stop codon. This avoids adding stop symbols into the protein sequence.

### Output:

The translated protein sequence is saved as:

```text
egfr_protein.fasta
```
---

## Step 7. Prepare the EGFR Kinase-Domain FASTA for AlphaFold2

The full EGFR protein is large. The notebook extracts the kinase-domain region for structure prediction.

The kinase domain is the region targeted by many EGFR inhibitors. Erlotinib binds near the ATP-binding pocket of this domain. Since the docking part focuses on erlotinib binding, the kinase domain is the most relevant region for this project.


### Region used

The notebook extracts residues:

```text
671–998
```

In Python, indexing starts from 0, so residue 671 corresponds to index 670:

```python
egfr_kinase_domain = egfr_full_protein[670:998]
```

### Output

The extracted kinase-domain sequence is saved as:

```text
EGFR_Project/data/egfr_kinase_domain_for_alphafold.fasta
```

This is the sequence that should be submitted to AlphaFold2 or ColabFold.

---

## Step 8. Display the AlphaFold2 Input Sequence

The notebook prints the kinase-domain FASTA sequence.

Before submitting a sequence to AlphaFold2 or ColabFold, it is important to verify that the correct sequence is being used. This step allows the user to copy and paste the FASTA sequence into AlphaFold2 or ColabFold.

- The sequence has a FASTA header beginning with `>`.
- The sequence contains only amino-acid letters.
- The sequence length matches the expected kinase-domain length.
- No DNA or RNA bases are accidentally being used as AlphaFold2 input.

---

## Step 9. Upload the AlphaFold2 Predicted PDB File

After running AlphaFold2 or ColabFold externally, the predicted PDB structure is uploaded into the notebook.

A PDB file contains three-dimensional atomic coordinates of a biological molecule. For a predicted protein structure, it stores the position of atoms in the protein chain.

The notebook again opens a Colab upload window:

```python
uploaded = files.upload()
```

The user selects the AlphaFold2/ColabFold predicted `.pdb` file.

The predicted PDB file is needed for visualization and project documentation. It represents the 3D structure predicted from the EGFR kinase-domain amino-acid sequence.

---

## Step 10. Visualize the AlphaFold2 Predicted Structure

The notebook uses `py3Dmol` to display the AlphaFold2 predicted EGFR kinase-domain structure.

The notebook:

1. Finds the uploaded predicted PDB file.
2. Reads the PDB structure.
3. Opens a 3D viewer.
4. Displays the protein as a cartoon model.

### Output

Protein structures are easier to understand visually than as coordinate files. Cartoon visualization helps show:

- Alpha helices
- Beta sheets
- Loops
- Overall domain fold

A 3D interactive protein model should appear inside the notebook. The user can rotate, zoom, and inspect the predicted structure.

<img width="800" height="500" alt="alphafold structure" src="https://github.com/user-attachments/assets/5b59300d-811e-4ff7-bb15-f863ca29efe1" />


---

## Step 11. Download an EGFR Crystal Structure for GNINA Docking

The notebook downloads the EGFR crystal structure with PDB ID:

```text
1M17
```
The 1M17 structure contains the EGFR kinase domain bound to erlotinib. This makes it useful for preparing:

- the receptor structure,
- the erlotinib ligand,
- the docking box center.

The downloaded file is saved as:

```text
EGFR_Project/structures/egfr_1m17.pdb
```

### Why this step comes after AlphaFold2

The corrected workflow first performs sequence-to-structure prediction using AlphaFold2, then performs docking with GNINA. This keeps the logic clear:

```text
Protein sequence → predicted structure → docking study
```

---

## Step 12. Separate Receptor and Ligand from the PDB File

Docking requires the receptor and ligand to be stored as separate files. GNINA needs a receptor file and ligand file. If the ligand remains inside the receptor PDB file, docking input preparation becomes incorrect. Separating receptor and ligand makes the docking setup clear and controlled.

From `egfr_1m17.pdb`, the notebook separates:

- Protein receptor atoms from `ATOM` records
- Erlotinib ligand atoms from `HETATM` records containing ligand code `AQ4`

### Output files

The receptor is saved as:

```text
EGFR_Project/structures/egfr_receptor.pdb
```

The extracted ligand is saved as:

```text
EGFR_Project/structures/erlotinib_from_pdb.pdb
```

---

## Step 13. Install GNINA

The notebook downloads GNINA and makes it executable. Before running docking, the notebook must confirm that GNINA was downloaded correctly and can run inside the Colab environment.

The notebook downloads the GNINA binary:

```bash
wget https://github.com/gnina/gnina/releases/download/v1.3.2/gnina.1.3.2 -O gnina
chmod +x gnina
```

Then it checks GNINA by running:

```bash
./gnina --help | head
```
---

## Step 14. Convert the Ligand into SDF Format

The extracted erlotinib ligand is converted from PDB format into SDF format using Open Babel. Ligands can be represented in different chemical file formats. SDF is commonly used for small molecules because it can store atom coordinates, bonding information, and molecular properties.

### Input file

```text
EGFR_Project/structures/erlotinib_from_pdb.pdb
```

### Output file

```text
EGFR_Project/structures/erlotinib.sdf
```

The notebook uses:

```bash
obabel erlotinib_from_pdb.pdb -O erlotinib.sdf --gen3d
```

---

## Step 15. Calculate the Docking Box Center

Docking must be performed inside a defined search region called a docking box. Erlotinib is already present in the 1M17 crystal structure, its position marks the known binding pocket. Using the ligand's average coordinate is a practical way to center the docking search box around the active site.

The notebook reads the coordinates of the extracted erlotinib ligand atoms and calculates the mean coordinate:

```python
center = coords.mean(axis=0)
```

This gives:

```text
center_x
center_y
center_z
```

### output

The notebook prints values like:

```text
center_x = ...
center_y = ...
center_z = ...
```

These values are passed directly into the GNINA docking command.

---

## Step 16. Run GNINA Docking

GNINA docks erlotinib into the EGFR receptor binding pocket.

### Inputs used

Receptor:

```text
EGFR_Project/structures/egfr_receptor.pdb
```

Ligand:

```text
EGFR_Project/structures/erlotinib.sdf
```

Docking box center:

```text
center_x, center_y, center_z
```

Docking box size:

```text
size_x = 20
size_y = 20
size_z = 20
```

### Output files

Docked ligand poses:

```text
EGFR_Project/docking_results/egfr_erlotinib_docked.sdf
```

GNINA log file:

```text
EGFR_Project/docking_results/gnina_log.txt
```

### What GNINA does

GNINA searches for possible orientations and conformations of the ligand inside the binding pocket. Each predicted ligand arrangement is called a **pose**. GNINA then scores the poses based on predicted binding quality.

---

## Step 17. View GNINA Docking Scores

The notebook displays the GNINA log file.

### Score columns

GNINA commonly reports values such as:

| Score | Meaning |
|---|---|
| Affinity | Predicted binding energy in kcal/mol |
| CNN score | Neural-network-based pose quality score |
| CNN affinity | Neural-network-predicted binding affinity |

### Interpret affinity

A more negative affinity value generally suggests stronger predicted binding. For example:

```text
-9.0 kcal/mol
```

is usually considered stronger than:

```text
-6.0 kcal/mol
```

However, docking scores are predictions. They should not be treated as experimental proof of binding.

---

## Step 18. Save Docking Scores as a CSV File

The notebook parses the GNINA log file and saves the scores into a CSV file.
## Docking Results

Molecular docking generated nine predicted binding poses. The docking affinity values are reported in kcal/mol, where more negative values generally indicate stronger predicted binding. CNN score and CNN affinity are additional neural-network-based scoring metrics used to estimate pose quality and binding strength.


### Output file

```text
EGFR_Project/docking_results/docking_scores.csv


| Mode | Affinity (kcal/mol) | CNN Score | CNN Affinity |
|---:|---:|---:|---:|
| 1 | -7.00 | -0.72 | 0.9207 |
| 2 | -7.22 | -0.90 | 0.9141 |
| 3 | -7.48 | -0.03 | 0.8995 |
| 4 | -6.40 | -0.87 | 0.8343 |
| 5 | -6.34 | -1.13 | 0.8194 |
| 6 | -7.28 | -0.55 | 0.7370 |
| 7 | -7.16 | -0.69 | 0.7227 |
| 8 | -6.66 | -0.62 | 0.6619 |
| 9 | -6.49 | -1.15 | 0.6589 |



```
### Interpretation

The best docking affinity was observed for **mode 3**, with a binding energy of **-7.48 kcal/mol**, suggesting it is the strongest predicted binding pose among the generated conformations. Modes **6**, **2**, and **7** also showed favorable binding affinities, with scores of **-7.28**, **-7.22**, and **-7.16 kcal/mol**, respectively.

Although mode 3 has the most negative binding affinity, mode 1 has the highest CNN affinity value (**0.9207**), followed closely by mode 2 (**0.9141**) and mode 3 (**0.8995**). This suggests that modes 1–3 are the most reliable poses overall when considering both docking energy and CNN-based scoring.

Overall, the docking results indicate that the ligand shows favorable predicted binding to the target protein, with the top-ranked poses falling around **-7.0 to -7.5 kcal/mol**. These poses may be selected for further visualization, interaction analysis, or molecular dynamics simulation.

---

## Step 19. Visualize the Docked EGFR–Erlotinib Complex

The final step visualizes the receptor and docked ligand together. Docking scores are useful, but visual inspection is also important. Visualization helps confirm whether the ligand is positioned inside the expected binding pocket.


The notebook loads:

- `egfr_receptor.pdb`
- `egfr_erlotinib_docked.sdf`

Then `py3Dmol` displays:

- the receptor as a cartoon model,
- the docked ligand as sticks.

<img width="800" height="500" alt="Gnina result" src="https://github.com/user-attachments/assets/f90868c9-0244-4f54-bc49-ff1384de6fce" />


---

## Interpreting the Results

### Sequence results

The DNA, mRNA, and protein files show the conversion from gene sequence to amino-acid sequence. The protein sequence is the biologically relevant input for AlphaFold2.

### AlphaFold2 results

The predicted PDB file represents the 3D fold of the EGFR kinase domain. If using ColabFold or AlphaFold2, confidence scores such as pLDDT may also be available. Higher confidence regions are generally more reliable than low-confidence regions.

### Docking results

GNINA produces docking poses and scores. The most important file for quick interpretation is:

```text
docking_scores.csv
```

A pose with stronger predicted affinity and good CNN score may be considered a better predicted binding pose. However, docking should be interpreted carefully. It is a computational prediction, not experimental validation.

---

## Important Notes and Limitations

1. This workflow is for educational and research training purposes.
2. The DNA input should be a coding sequence, not raw genomic DNA with introns.
3. AlphaFold2 predicts structure from sequence, but predicted structures may still need validation.
4. Docking scores are approximate and should not be treated as exact binding energies.
5. The ligand preparation in this notebook is simple. More advanced docking studies may require protonation-state assignment, energy minimization, receptor preparation, water handling, and validation against known binding poses.
6. The 1M17 crystal structure is used because it already contains erlotinib in the EGFR kinase-domain pocket.
7. Results may vary depending on software versions, random seeds, and computing environment.
8. This project does not prove drug effectiveness or clinical activity.

---

## References

- AlphaFold2: protein structure prediction method developed by DeepMind.
- ColabFold: simplified AlphaFold2/AlphaFold-Multimer workflow for faster structure prediction.
- GNINA: molecular docking software with convolutional neural network scoring.
- Biopython: Python library for biological sequence analysis.
- Open Babel: chemical file format conversion toolkit.
- RCSB Protein Data Bank: source of experimental protein structures, including EGFR structure 1M17.
- py3Dmol: 3D molecular visualization tool for notebooks.

---

## Project Summary

This project demonstrates a full computational biology pipeline starting from an EGFR DNA sequence and ending with EGFR-erlotinib docking results. The workflow follows the correct biological and computational order:

```text
DNA → mRNA → protein → EGFR kinase-domain FASTA → AlphaFold2 predicted structure → GNINA docking → docking score analysis → 3D visualization
```

