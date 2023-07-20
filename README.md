# WITCH - WeIghTed Consensus Hmm alignment
<!--
![visitors](https://visitor-badge.glitch.me/badge?page_id=c5shen.visitor-badge&left_color=blue&right_color=black))
-->

[![publication](https://img.shields.io/badge/Publication-Journal_of_Computational_Biology-green?style=for-the-badge)](https://doi.org/10.1089/cmb.2021.0585)

(C) Chengze Shen, Baqiao Liu

_Special thanks to Baqiao for providing the experimental GCM code!_

-----------------------------
News
-----------------------------
* An improved version of WITCH (developed by Baqiao Liu) is available at [WITCH-ng](https://github.com/RuneBlaze/WITCH-NG). It improves WITCH runtime considerably with the same alignment accuracy.

-----------------------------
Method Overview
-----------------------------
### Algorithm
WITCH is a new multiple sequence alignment (MSA) tool that combines techniques from [UPP](https://github.com/smirarab/sepp/blob/master/README.UPP.md) and [MAGUS](https://github.com/vlasmirnov/MAGUS). It aims to solve alignment problems particularly when input sequences contain fragments. The whole pipeline can be described as follows:
1. Given a set of unaligned sequences `S`, pick at most 1,000 "full-length" sequences to form a _backbone alignment_ `B` and a _backbone tree_ `T` (Full-length sequences refer to sequences of lengths that are within 25% of the median length).
2. Create an ensemble of HMMs (eHMM, see [UPP](https://github.com/smirarab/sepp/blob/master/README.UPP.md) for more details) from `B` and `T`.
3. For each remaining unaligned sequence, align it to high-ranked HMMs to obtain a set of weighted support alignments; then, merge the support alignments using Graph Clustering Merger (GCM, an alignment merger technique introduced in MAGUS).
4. Transitively add the merged alignment of each query to `B`, and report the final alignment on `S`.

For a more detailed explanation of the WITCH algorithm, please refer to the publication below:

#### Publication
1. Shen, Chengze, Minhyuk Park, and Tandy Warnow. “WITCH: Improved Multiple Sequence Alignment Through Weighted Consensus Hidden Markov Model Alignment.” Journal of Computational Biology, May 17, 2022. https://doi.org/10.1089/cmb.2021.0585.

### Note and Acknowledgement
- WITCH includes and uses:
    1. [MAGUS](https://github.com/vlasmirnov/MAGUS) (we use the Github version updated on April 5th 2021).
    2. [HMMER suites](http://hmmer.org/) (v3.1b2 - hmmbuild, hmmsearch, hmmalign).
    3. [UPP](https://github.com/smirarab/sepp/blob/master/README.UPP.md) (v4.5.1; we use only partial functionalities).
    4. [FastTreeMP](http://www.microbesonline.org/fasttree/FastTreeMP) (v2.1).
    5. [MAFFT](https://mafft.cbrc.jp/alignment/software/macportable.html) (macOS v7.490).
    6. [MCL](https://github.com/micans/mcl) (linux version from MAGUS; macOS version 21-257).


---------------------------
Installation
---------------------------
This section lays out necessary steps to do to run WITCH. We tested WITCH on the following systems:
* Red Hat Enterprise Linux Server release 7.9 (Maipo) with **Python 3.7.0**
* Ubuntu 18.04.6 LTS with **Python 3.7.6**, and Ubuntu 22.04 LTS with **Python 3.7.12**
* macOS _(x86 chip)_ Monterey 12.4 with **Python 3.9.13**

Now the program fully supports Linux and macOS systems (for at least the ones mentioned above). We provide necessary binary executables for both types of systems, but you can supplement your own by changing the paths in the `main.config` file. In cases of conflicting installations (e.g., different versions of MAFFT), please supplement with the version on your system.
If you experience any difficulty running WITCH, please contact Chengze Shen (chengze5@illinois.edu).

### Python version (REQUIRED!)
```
python>=3.7
```

### Requirements
```
cython>=0.29
configparser>=5.0.0
DendroPy>=4.4.0,<4.6.0
numpy>=1.15
psutil>=5.0
tqdm>=4.0.0
```

### Installation Steps
```bash
# 1. Install via GitHub repo
git clone https://github.com/c5shen/WITCH.git

# 2. Install all requirements
# If you do not have root access, use "pip3 install -r requirements.txt --user"
cd WITCH
pip3 install -r requirements.txt

# 3. Run setup.py to set up main.config. Please refer to default.config and use `-h` for additional information
#    Additionally, software binaries that are available in the user's environment will be prioritized for usage.
#    Use "-p false" to disable this priority.
python3 setup.py [-h]

# 4. Execute the WITCH python script with -h to see allowed commandline parameter settings
#    When running WITCH normally, if step 3 is not run, WITCH will automatically generate a "main.config" file
#    using the default "setup.py" settings.
python3 witch.py [-h]

```

----------------------------
Usage
----------------------------
General command to run WITCH:
```
python3 witch.py -i <unaligned sequence file> -d <output directory> -o <output filename>
```
**Default behavior**: WITCH will pick at most 1,000 sequences from the input around the median length as the backbone sequences. Then, it uses MAGUS to align the backbone sequences and FastTree2 to estimate a tree. It uses UPP decomposition strategy to generate an eHMM, and uses HMMSearch to calculate bit scores between HMMs and unaligned sequences. Bit scores are used to calculate weights, and each unaligned sequence is aligned to top `k=10` HMMs ranked by weights.

#### Use regular bit scores
By default, WITCH uses HMMSearch to obtain bit scores, and then uses bit scores to calculate weights between unaligned sequences and HMMs. To use bit scores instead of weights, run WITCH by the following command:
```bash
python3 witch.py -w 0 [...other parameters...]
```

#### Multi-processing
By default, WITCH uses all available cores on the machine. Users can choose the number of cores by the following command:
```bash
python3 witch.py -t <number of cpus> [...other parameters...]
```

To obtain the full list of parameters and options, please use `python3 witch.py -h` or `python3 witch.py --help`.

-------------------------
Examples
-------------------------
All the following examples can be found in the **examples/run.sh** bash script.
### Scenario A - unaligned sequences only
```bash
python3 witch.py -i examples/data/unaligned_all.txt -d scenarioA_output -o aligned.txt
```

### Scenario B - unaligned sequences only; using bit scores; using 10 HMMs to align a sequence
```bash
python3 witch.py -i examples/data/unaligned_all.txt -d scenarioB_output -o aligned.txt -w 0 -k 10
```

### Scenario C - backbone alignment available; backbone tree missing; query sequences available
```bash
python3 witch.py -b examples/data/backbone.aln.fasta -q examples/data/unaligned_frag.txt -d scenarioC_output -o aligned.txt
```

-------------------------
To-do
-------------------------
1. Optimize code struction in `gcmm/tree.py`, particularly for `decompose_tree(...)`.
1. Setting up checkpoint and allow resuming a job from checkpoints (to avoid re-running those paintstaking hmmsearch jobs!).
2. Checking the effect of large chunks vs. smaller chunks when the number of cores is small. Maybe submit many jobs of smaller chunks can improve overall runtime.
3. Improve logging to have some more intermediate output to `log.txt`.
1. (DONE) ~~Optimize how merging is done to reduce the output file size (need to do a similar task as UPP where insertion columns are marked as lower cases and squeezed together.~~
2. (DONE) ~~Optimize I/O so that there are fewer intermediate file writeouts and readins.~~
3. (Need manual fix from users) ~~Support for arm64 systems (mainly an issue for FastTreeMP).~~
