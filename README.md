# VIGA
De novo Viral Genome Annotator

VIGA is a script written in Python 2.7 that annotates viral genomes automatically (using a de novo algorithm) and predict the function of their proteins using BLAST and HMMER.

## REQUIREMENTS:

Before using this script, the following Python modules and programs should be installed:

* Python modules:
	- BCBio.GFF (https://github.com/chapmanb/bcbb/tree/master/gff)
	- Biopython (Bio module; Cock et al. 2009)
	- Numpy (https://github.com/numpy/numpy)
	- Scipy (https://github.com/scipy/scipy)

* Programs:
	- GNU Parallel (Tange 2011): it is used to parallelize HMMER. The program is publicly available at https://www.gnu.org/software/parallel/ under the GPLv3 licence.
	- LASTZ (Harris 2007): it is used to predict the circularity of the contigs. The program is publicly available at https://github.com/lastz/lastz under the MIT licence.
	- Prodigal (Hyatt et al. 2010): it is used to predict the ORFs. When the contig is smaller than 20,000 bp, MetaProdigal (Hyatt et al. 2012) is automatically activated instead of normal Prodigal. This program is publicly available at https://github.com/hyattpd/prodigal/releases/ under the GPLv3 licence.
	- BLAST+ (Camacho et al. 2008): it is used to predict the function of the predicted proteins according to homology. This suite is publicly available at ftp://ftp.ncbi.nlm.nih.gov/blast/executables/blast+/LATEST/ under the GPLv2 licence. Databases are available at ftp://ftp.ncbi.nlm.nih.gov/blast/db/
	- DIAMOND (Buchfink et al. 2015): it is used to predict the function of proteins according to homology when "--noblast" parameter is used. This program is publicly available at https://github.com/bbuchfink/diamond under the GPLv3 licence. Databases must be created from FASTA files according to their instructions before running.
	- HMMER (Finn et al. 2011): it is used to predict the function of the predicted proteins according to Hidden Markov Models. This suite is publicly available at http://hmmer.org/ under the GPLv3 licence. Databases must be in FASTA format and examples of potential databases are UniProtKB (ftp://ftp.uniprot.org/pub/databases/uniprot/current_release/knowledgebase/complete/uniprot_trembl.fasta.gz) or PFAM (http://ftp.ebi.ac.uk/pub/databases/Pfam/current_release/Pfam-A.fasta.gz).
	- INFERNAL (Nawrocki and Eddy 2013): it is used to predict ribosomal RNA in the contigs when using the RFAM database (Nawrocki et al. 2015). This program is publicly available at http://eddylab.org/infernal/ under the BSD licence and RFAM database is available at ftp://ftp.ebi.ac.uk/pub/databases/Rfam/
	- ARAGORN (Laslett and Canback 2004): it is used to predict tRNA sequences in the contig. This program is publicly available at http://mbio-serv2.mbioekol.lu.se/ARAGORN/ under the GPLv2 licence.
	- PILERCR (Edgar 2007): it is used to predict CRISPR repeats in your contig. This program is freely available at http://drive5.com/pilercr/ under a public licence.
	- Tandem Repeats Finder (TRF; Benson 1999): it is used to predict the tandem repeats in your contig. This program is freely available at https://tandem.bu.edu/trf/trf.html under a custom licence.
	- Inverted Repeats Finder (IRF; Warburton et al. 2004): it is used to predict the inverted repeats in your contig. This program is freely available at https://tandem.bu.edu/irf/irf.download.html under a custom licence.

Although you can install the programs manually, we strongly recommend the use of the Docker image to create an environment for VIGA. The link to the Docker image is https://hub.docker.com/r/vimalkvn/viga/

However, you will need to download the databases for BLAST, HMMER, and INFERNAL:
* BLAST DBs: https://ftp.ncbi.nlm.nih.gov/blast/db/
* BLAST FASTA (DIAMOND): https://ftp.ncbi.nlm.nih.gov/blast/db/FASTA
* RFAM (INFERNAL): http://ftp.ebi.ac.uk/pub/databases/Rfam/
* UniProtKB (HMMER): ftp://ftp.uniprot.org/pub/databases/uniprot/current_release/knowledgebase/complete/uniprot_trembl.fasta.gz

To use this pipeline, it is highly recommended the use of the protein databases "nr" for BLAST and DIAMOND and "UniProt_TREMBL" for HMMER. Moreover, before running the "ultrafast" mode, you need to convert the FASTA file to the DIAMOND DB format using the following command:

	diamond makedb --in nr -d nr

When using this program, you must cite their use:

	González-Tortuero E, Sutton TDS, Velayudhan V, Shkoporov AN, Draper LA, Stockdale SR, Ross RP, Hill C (2018) VIGA: a sensitive, precise and automatic de noco VIral Genome Annotator. bioRxiv 277509; doi: https://doi.org/10.1101/277509 

## PARAMETERS:

The program has the following two types of arguments:

### Mandatory parameters:

<table>
<tr><td>--input FASTAFILE</td><td>Input file as a nucleotidic FASTA file. It can contains multiple sequences (e.g. metagenomic contigs)</td></tr>
<tr><td>--rfamdb RFAMDB</td><td>RFAM database that will be used for the ribosomal RNA prediction.</td></tr>
<tr><td>--modifiers TEXTFILE</td><td>Input file as a plain text file with the modifiers per every FASTA header according to SeqIn (https://www.ncbi.nlm.nih.gov/Sequin/modifiers.html). All modifiers must be written in a single line and are separated by a single space character. No space should be placed besides the = sign. For example: [organism=Serratia marcescens subsp. marcescens] [sub-species=marcescens] [strain=AH0650_Sm1] [moltype=DNA] [tech=wgs] [gcode=11] [country=Australia] [isolation-source=sputum]. This line will be copied and printed along with the record name as the definition line of every contig sequence.</tr></td>
</table>

### Advanced parameters:

<table>
<tr><td>--readlength INT</td><td>Read length for the circularity prediction (default: 101 bp)</td></tr>
<tr><td>--windowsize INT</td><td>Window length used to determine the origin of replication in circular contigs according to the cumulative GC skew(default: 100 bp)</td></tr>
<tr><td>--slidingsize INT</td><td>Sliding window length used to determine the origin of replication in circular contigs according to the cumulative GC skew(default: 10 bp)</td></tr>
<tr><td>--out OUTPUTNAME</td><td>Name of the outputs files without extensions, as the program will add them automatically. By default, the program will use the input name as the output.</td></tr>
<tr><td>--locus STRING</td><td>Name of the contigs. If the input is a multiFASTA file, please put a general name as the program will add the number of the contig at the end of the name. By default, the name of the contigs will be "LOC".</td></tr>
<tr><td>--threads INT</td><td>Number of threads/CPUs. By default, the program will use 1 CPU.</td></tr>
<tr><td>--gff</td><td>Printing the output as a General Feature Format (GFF) version 3. It is a flat table file with contains 9 columns of data (see http://www.ensembl.org/info/website/upload/gff3.html for more information). By default, the program will not print the GFF3 file (--gff False).</td></tr>
<tr><td>--blastdb BLASTDB</td><td>BLAST database that will be used for the protein function prediction. The database MUST be for amino acids. This is only mandatory if the "ultrafast" mode is not active</td></tr>
<tr><td>--diamonddb DIAMONDDB</td><td>DIAMOND database that will be used for the protein function prediction. The database MUST be for amino acids. This is only mandatory when "ultrafast" mode is active</td></tr>
<tr><td>--blastevalue FLOAT</td><td>BLAST e-value threshold. By default, the threshold will be 1e-05.</td></tr>

<tr><td>--nohmmer</td><td>Running the program without using PHMMER to predict protein function. In this case, the program will be as fast as Prokka (Seemann 2014) but the annotations will not be accurate. By default, this program had this flag disabled.</td></tr>
<tr><td>--noblast</td><td>Running the program replacing BLAST by DIAMOND. In this case, the program will be fast but the annotations will not be accurate. By default, this program had this flag disabled.</td></tr>
<tr><td>--hmmdb HMMDB</td><td>PHMMER Database that will be used for the protein function prediction according to Hidden Markov Models. In this case, HMMDB must be in FASTA format (e.g. UniProt). This parameter is mandatory if the "--fast" option is disabled. "</td></tr>
<tr><td>--hmmerevalue FLOAT</td><td>PHMMER e-value threshold. By default, the threshold is 1e-03.</td></tr>
<tr><td>--typedata BCT|CON|VRL|PHG</td><td>GenBank Division: One of the following codes:
<table>
<tr><td>BCT</td><td>Prokaryotic chromosome</td></tr>
<tr><td>VRL</td><td>Eukaryotic/Archaea virus</td></tr>
<tr><td>PHG</td><td>Phages</td></tr>
<tr><td>CON</td><td>Contig</td></tr>
</table>
By default, the program will consider every sequence as a contig (CON)</td></tr>
<tr><td>--gcode NUMBER</td><td>Number of GenBank translation table. At this moment, the available options are:
<table>
<tr><td>1</td><td>Standard genetic code [Eukaryotic]</td></tr>
<tr><td>2</td><td>Vertebrate mitochondrial code</td></tr>
<tr><td>3</td><td>Yeast mitochondrial code</td></tr>
<tr><td>4</td><td>Mycoplasma/Spiroplasma and Protozoan/mold/coelenterate mitochondrial code</td></tr>
<tr><td>5</td><td>Invertebrate mitochondrial code</td></tr>
<tr><td>6</td><td>Ciliate/dasycladacean/Hexamita nuclear code</td></tr>
<tr><td>9</td><td>Echinoderm/flatworm mitochondrial code</td></tr>
<tr><td>10</td><td>Euplotid nuclear code</td></tr>
<tr><td>11</td><td>Bacteria/Archaea/Phages/Plant plastid</td></tr>
<tr><td>12</td><td>Alternative yeast nuclear code</td></tr>
<tr><td>13</td><td>Ascidian mitochondrial code</td></tr>
<tr><td>14</td><td>Alternative flatworm mitochondrial code</td></tr>
<tr><td>16</td><td>Chlorophycean mitochondrial code</td></tr>
<tr><td>21</td><td>Trematode mitochondrial code</td></tr>
<tr><td>22</td><td>Scedenesmus obliquus mitochondrial code</td></tr>
<tr><td>23</td><td>Thraustochytrium mitochondrial code</td></tr>
<tr><td>24</td><td>Pterobranquia mitochondrial code</td></tr>
<tr><td>25</td><td>Gracilibacteria and Candidate division SR1</td></tr>
<tr><td>26</td><td>Pachysolen tannophilus nuclear code</td></tr>
<tr><td>27</td><td>Karyorelict nuclear code</td></tr>
<tr><td>28</td><td>Condylostoma nuclear code</td></tr>
<tr><td>29</td><td>Mesodinium nuclear code</td></tr>
<tr><td>30</td><td>Peritrich nuclear code</td></tr>
<tr><td>31</td><td>Blastocrithidia nuclear code</td></tr>
</table>
By default, the program will use the translation table no. 11</td></tr>
<tr><td>--mincontigsize INT</td><td>Minimum contig length to be considered in the final files. By default, the program only consider from 200 bp.</td></tr>
<tr><td>--idthr FLOAT</td><td>Identity threshold to consider that a protein belong to a specific hit. By default, the threshold is 50.0 %</td></tr>
<tr><td>--coverthr FLOAT</td><td>Coverage threshold to consider that a protein belong to a specific hit. By default, the threshold is 50.0 %</td></tr>
<tr><td>--diffid FLOAT (>0.01)</td><td>Max allowed difference between the ID percentages of BLAST and HMMER. By default, the allowed difference is 5.00 % and we do not recommended to change such value.</td></tr>
<tr><td>--minrepeat INT</td><td>Minimum repeat length for CRISPR detection (Default: 16)</td></tr>
<tr><td>--maxrepeat INT</td><td>Maximum repeat length for CRISPR detection (Default: 64)</td></tr>
<tr><td>--minspacer INT</td><td>Minimum spacer length for CRISPR detection (Default: 8)</td></tr>
<tr><td>--maxspacer INT</td><td>Maximum spacer length for CRISPR detection (Default: 64)</td></tr>
<tr><td>--blastexh</td><td>Use of exhaustive BLAST to predict the proteins by homology according to Fozo et al. (2010). In this case, the search will be done using a word size of 2, a gap open penalty of 8, a gap extension penalty of 2, the PAM70 matrix instead of the BLOSUM62 and no compositional based statistics. This method is more accurate to predict the functions of the proteins but it is slower than BLAST default parameters. By default, exhaustive BLAST is disabled.</td></tr>
</table>

## Examples
An example of execution (using BLAST and HMMER) is:

	python VIGA.py --input eukarya.fasta --blastdb databases/blast/nr/nr --hmmdb databases/UniProt/uniprot_trembl.fasta --rfamdb databases/rfam/Rfam.cm --gcode 1 --out eukarya_BENCHMARK --modifiers ../modifiers.txt --threads 10

Another example (but this time using BLAST but not HMMER - "fast mode") is:

	python VIGA.py --input bacteria.fasta --blastdb databases/blast/nr/nr --nohmmer --rfamdb databases/rfam/Rfam.cm --out bacteria_BENCHMARK --modifiers ../modifiers.txt --threads 10
	
Finally, an example using DIAMOND and not HMMER ("ultrafast mode") is:

	python VIGA.py --input archaea.fasta --noblast --diamonddb databases/diamond/nr --nohmmer --rfamdb databases/rfam/Rfam.cm --out archaea_BENCHMARK --modifiers ../modifiers.txt --threads 10

## Galaxy wrapper
VIGA can be integrated into [Galaxy](https://galaxyproject.org) using the wrapper included in this repository.

### Requirements

[Docker](https://www.docker.com) should first be installed and working on the
server where this Galaxy instance is setup. The user running Galaxy should be
part of the **docker** user group.

### Installation

1. Download or clone this repository (as a submodule) in the **tools**
   directory of the Galaxy installation.

2. Update **config/tool_conf.xml** to add the VIGA wrapper in a relevant section of the tool panel. For example, "Annotation".

		<section id="annotation" name="Annotation">
			<tool file="viga/wrapper.xml" />
		</section>

3. Copy (or update the file if it is already present) the included
   **tool_data_table_conf.xml.sample** file to
   **config/tool_data_table_conf.xml**.

		<!-- VIGA databases -->
		<tables>
		    <table name="viga_blastdb" comment_char="#">
			<columns>value, dbkey, name, path</columns>
			<file path="tool-data/viga_blastdb.loc" />
		    </table>
		    <table name="viga_diamonddb" comment_char="#">
			<columns>value, dbkey, name, path</columns>
			<file path="tool-data/viga_diamonddb.loc" />
		    </table>
		    <table name="viga_rfamdb" comment_char="#">
			<columns>value, dbkey, name, path</columns>
			<file path="tool-data/viga_rfamdb.loc" />
		    </table>
		    <table name="viga_hmmdb" comment_char="#">
			<columns>value, dbkey, name, path</columns>
			<file path="tool-data/viga_hmmdb.loc" />
		    </table>
		</tables>

4. Copy the **.loc.sample** files from **viga/tool-data** to **galaxy/tool-data**
   and rename them as **.loc**. For example:

		viga_blastdb.loc.sample -> viga_blastdb.loc

5. Update database paths in .loc files
   Edit the following files in the **tool-data** directory and add paths to
   corresponding databases

   * viga_blastdb.loc
   * viga_diamonddb.loc
   * viga_rfamdb.loc
   * viga_hmmdb.loc


6. Create or update the Galaxy job configuration file

   If the file **config/job_conf.xml** does not exist, create it by copying the
   template **config/job_conf.xml.sample_basic** in the Galaxy directory. Then
   add a Docker destination for viga. Change ``/data/databases`` under
   ``docker_volumes`` to the location where your databases are stored. Here is
   an example:

		<?xml version="1.0"?>
		<!-- A sample job config that explicitly configures job running the way it is configured by default (if there is no explicit config). -->
		<job_conf>
		    <plugins>
			<plugin id="local" type="runner" load="galaxy.jobs.runners.local:LocalJobRunner" workers="4"/>
		    </plugins>
		    <handlers>
			<handler id="main"/>
		    </handlers>
		    <destinations default="local">
			<destination id="local" runner="local"/>
			<destination id="docker" runner="local">
				<param id="docker_enabled">true</param>
				<param id="docker_sudo">false</param>
				<param id="docker_auto_rm">true</param>
				<param id="docker_volumes">$defaults,/data/databases:ro</param>
			</destination>
		    </destinations>
		    <tools>
		      <tool id="viga" destination="docker"/>
		    </tools>
		</job_conf>


7. **Restart Galaxy**. The tool will now be ready to use.


## HISTORY OF THE SOURCE CODE:

* v 0.10.4 - Fixed error when the module BCBio is invoked. Now it is required the module BCBio.GFF instead of the BCBio one.
* v 0.10.3 - New output: all protein sequences per contig.
* v 0.10.1 - Fixed error when the start coordinate of a gene is equal to one. In these cases, genes were annotated as if they started in the position zero (which it has no biological logic). Now, the program should be able to deal with these genes, annotate them from the position 1. Moreover, added new terms to reduce all non-informative protein descriptions before running the decision tree.
* v 0.10.0 - Added the prediction of the origin and terminus of replication for circular contigs based on the cumulative GC skew (based on the iRep software - Brown et al (2016)). After detecting the origin coordinate, the chromosome is realigned from the origin. As a consequence of that, two new parameters ("--windowsize" and "--slidingsize") were added to determine the window size and the sliding window size respectively. Moreover, fixed error with the start position of the genes in the GenBank files, which were not related to the amino acid sequences and made that the sequence length was not multiple of three. Finally, added "/locus_tag" in the putative genes in the GenBank files
* v 0.9.1 - Fixed a bug in creation of logfile
* v 0.9.0 - Improved the BLAST/DIAMOND and HMMER parsers to reduce all non-informative protein descriptions (e.g. "hypothetical protein", "ORF") before running the decision tree algorithm. Additionally, a new output file (logfile.txt) is generated to harbour the information about the old contig names and the new ones (generated by the program).
* v 0.8.2 - Fixed issue with DIAMOND when there is no protein sequence as input.
* v 0.8.1 - The program is able to deal with tmRNA sequences in a proper way. There were an error due to the "(Permuted)" flag in ARAGORN files in some cases. Additionally, the name of the "--fast" and "--ultrafast" parameters were changed to "--nohmmer" and "--noblast" as their descriptions are more accurate.
* v 0.8.0 - Added the "--ultrafast" parameter. In this case, DIAMOND (Buchfink et al. 2015) will be launch to predict protein function according to homology instead of BLAST. It is faster than the "fast" mode but the sensitivity of the annotations will not be the highest.
* v 0.7.1 - Fixed error on the "--fast" parameter. All proteins that had no hits in BLAST analyses were not parsed properly. By now, these are identified as "Hypothetical proteins" in all files.
* v 0.7.0 - Added the "--fast" parameter. In this case, the program will launch BLAST (but not PHMMER) to annotate protein function. In this case, the program will be as fast as Prokka (Seemann 2014) but the annotations will not be accurate. As a consequence of this new parameter, the "--hmmdb" parameter is only mandatory when this flag is NOT used (as by default).
* v 0.6.2 - Removed the "--noparallel" parameter. After doing time benchmarks to test the speed of BLAST and HMMER when they are run using the multithreading option and as a parallel program, we found that BLAST tends to be faster using multithreading option while HMMER had the opposite behavior. For that, we decided to consider only the parallelization of HMMER and to run BLAST using multiple threads. 
* v 0.6.1 - Fixed issue with parallel HMMER (the program tend to take all available CPUs independently of the parsed arguments) and with the BLAST/HMMER decision trees (typos).
* v 0.6.0 - Replaced HHSUITE by HMMER 3.1 to predict protein function according to Hidden Markov Models. In a recent benchmark (as well as internal ones), we found that HHPred tends to be the slowest program to predict protein function (compared with PHMMER and BLASTP). Additionally, HMMER had a high accuracy when proteins are annotated (Saripella et al. 2016). Moreover, it has the advantage that the databases must be in FASTA format (such UniProt and, even, PFAM), which it is a standard format. For all these reasons, we replaced HHSUITE by HMMER 3.1. Additionally, fixed small issues related to the GenBank file (omission of the contig topology as well as the name of the locus).
* v 0.5.0 - Implemented PILER-CR to predict CRISPR repeats regions. Additionally, fixed errors in the rRNA prediction and inverted and tandem repeats.
* v 0.4.0 - Replaced RNAmmer v 1.2. by INFERNAL 1.1 + RFAM to predict rRNA in the contigs. In this case, you must specify where you have downloaded the RFAM database using the "--rfamdb" option.
* v 0.3.0 - Implemented RNAmmer v 1.2 to predict rRNA in the contigs. If such program is able to predict ribosomal genes, a warning is printed (as viral sequences do not have ribosomal genes).
* v 0.2.0 - Added parallelization of BLAST and HHSUITE. To do that, GNU Parallel (Tange 2011) is required. To disable this option, run the program with the "--noparallel" option.
* v 0.1.0 - Original version of the program.

## REFERENCES:

	- Benson G (2008) Tandem repeats finder: a program to analyze DNA sequences. Nucleic Acids Research 27: 573–80.
	- Brown CT, Olm MR, Thomas BC, Banfield JF (2016) Measurement of bacterial replication rates in microbial communities. Nature Biotechnology 34: 1256-63.
	- Buchfink B, Xie C, Huson DH (2015) Fast and sensitive protein alignment using DIAMOND. Nature Methods 12: 59-60.
	- Camacho C, Coulouris G, Avagyan V, Ma N, Papadopoulos J, Bealer K, Madden TL (2008) BLAST+: architecture and applications. BMC Bioinformatics 10: 421.
	- Edgar RC (2007) PILER-CR: fast and accurate identification of CRISPR repeats. BMC Bioinformatics 8:18.
	- Finn RD, Clements J, Eddy SR (2011) HMMER web server: interactive sequence similarity searching. Nucleic Acids Research 39: W29-37.
	- Fozo EM, Makarova KS, Shabalina SA, Yutin N, Koonin EV, Storz G (2010) Abundance of type I toxin-antitoxin systems in bacteria: searches for new candidates and discovery of novel families. Nucleic Acids Research 38: 3743-59.
	- Harris RS (2007) Improved pairwise alignment of genomic DNA. Ph.D. Thesis, The Pennsylvania State University. 
	- Hyatt D, Chen GL, Locascio PF, Land ML, Larimer FW, Hauser LJ (2010) Prodigal: prokaryotic gene recognition and translation initiation site identification. BMC Bioinformatics 11: 119.
	- Hyatt D, Locascio PF, Hauser LJ, Uberbacher EC (2012) Gene and translation initiation site prediction in metagenomic sequences. Bioinformatics 28: 2223-30.
	- Laslett D, Canback B (2004) ARAGORN, a program to detect tRNA genes and tmRNA genes in nucleotide sequences. Nucleic Acids Research 32, 11–16.
	- Nawrocki EP, Eddy SR (2013) Infernal 1.1: 100-fold faster RNA homology searches. Bioinformatics 29: 2933-35.
	- Nawrocki EP, Burge SW, Bateman A, Daub J, Eberhardt RY, Eddy SR, Floden EW, Gardner PP, Jones TA, Tate J, Finn RD (2013) Rfam 12.0: updates to the RNA families database. Nucleic Acids Research 43: D130-7.
	- Saripella GV, Sonnhammer EL, Forslund K (2016) Benchmarking the next generation of homology inference tools. Bioinformatics 32: 2636-41.
	- Seemann T (2014) Prokka: rapid prokaryote genome annotation. Bioinformatics 30: 2068-9.
	- Tange O (2011) GNU Parallel - The Command-Line Power Tool. ;login: The USENIX Magazine 36:42-7.
	- Warburton PE, Giordano J, Cheung F, Gelfand Y, Benson G (2004) Inverted repeat structure of the human genome: The X-chromosome contains a preponderance of large, highly homologous inverted repeats that contain testes genes. Genome Research 14: 1861-9.
