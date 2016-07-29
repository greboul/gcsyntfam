# README of gcsyntfam tool

## When is this tool usefull?
You'll find usefull this tool when you will want to study the genomic context of a set of proteins from UniProtKB in order to group together the proteins with a similar genomic context within this set.

## What is it doing?
This tool is divided into 2 main steps:
* first, retrieving the genomic context of each protein from the set
* second, calculate the synteny between all the proteins according to blastP homology

## What to do before running it for the first time?
### Check or install the requirements
In order to use this tool you will need multiple tools or version of tools:
* python 2.7 or latest python 2 (does not work with python 3 !)
* biopython 1.66
* micSynteny
* jdk 1.6
* blast+ 2.2
* DBMS MySQL 

### Setting profile
In order to access the database properly during the process, you'll have to set the multiple environnement variable in the file profileDB.sh and then sourcing it.

### Create the database
* You will have now to create the database with the _SAME_ name that you filled in the profileDB.sh manually.
* Then you will have to load the file tablesForSyntenyDatabase.sql to create the tables into the database.

## Current usage
### Preparing files and folders for an UniProtKB list of accession identifier using _gcsyntfam\_families\_preparation.sh_
This script will create the directory set by the user and an other directory with the same name but with the suffix \_other\_files.
In the first one, 2 files and 2 folders are created :
* GCsize.txt: file with the genomic context size that the user ask for (third argument)
* GO_id.list: list of the uniq identifier that replace the UniProtKB identifier for each genomic object (GO)
* Nodes: folder that contains one file for each protein corresponding to its genomic context (genes and informations)
* Proteomes: folder that contains proteic fasta files for each genomic context (every proteic sequence for every genes involved in the genomic context of a protein)     
   
The folder "\_other\_files" contains additional informations (2 files and 2 folders too):    
* CPD_EMBLid_TAXid_PBid_GOid.txt: file containing correspondances between UniProtKB identifier (PB), EMBL assembly (EMBL), taxonomy identifier (TAX) and the Genomic Object used as a uniq key.
* CPD_PB_EMBL.txt: same file but only correspondances between UniProtKB identifier (PB), EMBL assembly (EMBL).
* embl_assemblies: folder that contains all the assemblies downloaded
* log: log folder

execute _gcsyntfam\_families\_preparation.sh_ \<PB\_list file\> \<Nbcol file\> \<size GC\> \<synteny directory\>   
*\<PB\_list file\>: the file containing UniProtKB identifiers in its _FIRST_ column   
*\<Nbcol file\>: integer corresponding to the number of columns in the PB\_list file (ecxept the first column, other columns can be used to store metadata)   
*\<size GC\>: integer corresponding to how many genes before and after the gene of interest you want to keep (). Warning, this integer should be >= 2 to obtain at least 5 genes (included the one of interest) in total to compare for each protein.   
*\<synteny directory\>:  the name of the directory with only characters from [A-Za-z0-9-] allowed in which the preparation will be done   
This step can be run multiple times with multiple sets of proteins in order to compare within the sets but also between the sets the genomic contexts of the proteins of interest.

### Setting a project
The tables in the database are based on a hierarchical scheme to help the user to structure his work. Indeed, for a Project, you can run multiple Analyses with different parameters for the homology during the blastP. A project should correspond to a set or multiple set of proteins. The analyses will be stored during the process but a Project have to be created manually using the script setProject.sh. This one will print the identifier created for the Project that is required for the following script.

### Preparing the commands to run and load the syntenies using _gcsyntfam\_commands\_preparation.sh_
This script will create the command lines to compute the syntenies and to load the results into the database. It will create an Analyse into the database with a link to the Project and a folder with the analyse ID (integer). In this folder there will be:
* all\_GO\_id.list.prefixed: contains all the Genomic Object identifiers from every source directory inputed. Every GO is prefixed with its folder name   
* computeSynteny.commands: file containing the commands to compute the syntenies    
* computeSynteny.load.commands: file containing the loading commands for the results  
* recap\_analyse\_X.txt: files that summarized the analyse, X is the integer corresponding to the analyse unique identifier    

* \<sourceDIR\_list\>: list of the directories to include into the analyse, comma separated (at least one)    
* \<P\_id\>: unique identifier of the Project from the database (P_id from table Project) generated by setProject.sh when the project was created    
* \<minCoverage\>: minimal coverage (between 0 and 1) by the alignement between 2 sequences on the smallest sequences    
* \<'id' or 'sim'\>: string 'id' or 'sim' to either choose identity or similarity to verify homology using the blastP results     
* \<%identity or %similarity\>: the percentage to use to check homology on the previous argument on the blastP results     
* \<Synteny gap\>: number of gaps allowed before breaking a synteny, this integer should be less than 10    
* \<delete files: yes/no\>: to delete or not 'roles' and '.res' files. Those files are used to create the .res.DB and SNTON.DB files that contain the results to store in the databse.    

### Run the commands to calculate the syntenies
Everything is prepared, so go on and calculate the syntenies!!    
run the following command :     
bash computeSynteny.commands    
Or use something to parallelized this step
### Run the loading commands to add the results to the database
Now you can load your results into the databse with this command:    
bash computeSynteny.load.commands     
WARNING: this step can not be paralelized!!

## What to do next?
In order to visualyze the different genomic context groups, a clustering is required.
