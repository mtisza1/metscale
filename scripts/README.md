# Database Query Tool (DQT)

## Table of Contents
* [Workflow Overview](#Workflow-Overview)
* [Required Files](#Required-Files)
* [Workflow Execution](#Workflow-Execution)
* [Custom Database](#Adding-a-Custom-Database)
* [Additional Information](#Additional-Information)

## Workflow Overview 
The Databse Query Tool is used to compare the contents of the reference databases used by various taxonomic classification tools. Specifically, since NCBI taxonomy is constantly being updated and different taxonomic classification tools use different databases, not all tools have the same reference organisms in their database. Thus, comparing outputs from one tool to another requires accounting for differences in the presence of organisms in their respective reference databases. If a taxonomic classification tool does not report an expected species in a metagenome, the DQT allows users to quickly query whether or not that species was present in the tool's reference database. The DQT allows the user to input one or more taxon IDs and output a list of the databases that contain that taxon ID or its ancestor. While taxons from any level can be queried, this tool was specifically designed to work with species-level taxons. 

![](https://github.com/signaturescience/metagenomics-wiki/blob/master/documentation/figures/DQT%20v1.png)

## Required Files
If you have not already, you will need to clone the MetScale repository and activate your metscale environment (see [Install](https://github.com/signaturescience/metscale/wiki/02.-Install)) before proceeding:

```sh
[user@localhost ~]$ source activate metscale 

(metscale)[user@localhost ~]$ cd metscale/scripts

```

### Input Files

If you ran the MetScale installation correctly, the following files and directories should be present in the `metscale/scripts` directory.

| File Name | File Size | MD5 Checksum |
| ------------- | ------------- | ------------- |
| `query_tool.py` | `74 KB` | `6281df76ea6e5e1b3546af5f0cd6ad2b` |
| `testtax.txt` | `188 B` | `195cd9131bcbcfcc1bea2aa1793a5740` |
| `containment_dict_.json.gz` | `14 MB` | `ab3dc6fe977be177554c1ab7919036c9` |
| `doc/` | `4 KB` | `directory` |
| `example_input_files/` | `4 KB` | `directory` | 

If you are missing any of these files, you should re-clone the MetScale repository, as per instructions in [Install](https://github.com/signaturescience/metscale/wiki/02.-Install). 

 
## Workflow Execution
### Quick Start:
After cloning the MetScale repository, some configuration is necessary before use. It can be done automatically using default settings by running the command:
```
python3 query_tool.py --setup
```
That will populate the setting `working_folder` in the default config file with the home folder of the DQT. Following that, the tool should be ready for use.

### Detailed Settings
The `--setup` command will automatically set the three important paths the DQT needs to run:
* 1) The repository of taxon coverage information for the various MetScale tools
* 2) The full reference taxonomy maintained by NCBI. 
* 3) The working folder for any outputs that are provided. 
These are the first three settings listed in the config file:
```
[paths]
working_folder = 
path_to_containment_file = ${working_folder}/containment_dict.json
path_to_ncbi_taxonomy_nodes = ${working_folder}/ncbi_taxonomy/nodes.dmp
```
The command `--setup` first sets the value of `working_folder` in this file to be the path to the scripts folder containing the DQT (by default this is `metascale/scripts`). After this, it creates a folder for the NCBI taxonomy, then downloads and extracts the needed `nodes.dmp` to `path_to_ncbi_taxonomy_nodes`. Finally it decompresses the file `containment_dict.json.gz` to create the `containment_dict.json` which contains metadata about the taxa in all of the various databases in a single json file.

## Usage 

### Taxon ID Querying

The default usage of the tool is to give one or more taxon IDs and output a text-based report showing which databases contain that taxon ID. The output goes to the console by default but can optionally be directed to a file. The default queried databases include several of the tools in the MetScale Taxonomic Classification Workflow, as well as versions of the NCBI RefSeq Database. The full list of databases is below.

|Tool|Database Name|Source|
|:---|:---|:---|
|RefSeq|`RefSeq_v98`|[NCBI RefSeq FTP](https://ftp.ncbi.nlm.nih.gov/refseq/release/release-catalog/archive/)|
|Kraken2|`minikraken2_v2_8GB_201904_UPDATE`|[Kraken2: minikraken2_v2 DB](https://genome-idx.s3.amazonaws.com/kraken/minikraken2_v2_8GB_201904.tgz) |
|KrakenUniq|`minikraken_20171019_8GB`|[Kraken1: minikraken_8GB `seqid2taxid.map`](https://ccb.jhu.edu/software/kraken/dl/seqid2taxid.map)|
|Kaiju|`kaiju_db_nr_euk`|(corresponds to [Kaiju NCBI *nr+euk* DB](http://kaiju.binf.ku.dk/database/kaiju_db_nr_euk_2019-06-25.tgz))|
|GenBank|`NCBI_nucl_gb`|[NCBI accn2taxid (nucl_gb)](https://ftp.ncbi.nlm.nih.gov/pub/taxonomy/accession2taxid/nucl_gb.accession2taxid.gz)|
|GenBank (WGS/TSA)|`NCBI_nucl_wgs`|[NCBI accn2taxid (nucl_wgs)](https://ftp.ncbi.nlm.nih.gov/pub/taxonomy/accession2taxid/nucl_wgs.accession2taxid.gz)|
|MetaPhlAn2|`metaphlan3`|[MetaPhlAn3 Google Drive](https://drive.google.com/drive/folders/1_HaY16mT7mZ_Z8JtesH8zCfG9ikWcLXG)|
|MTSv|`MTSV_Oct-28-2019`|[MTSv Complete Genome DB](https://rcdata.nau.edu/fofanov_lab/Compressed_MTSV_database_files/complete_genome.tar.gz)|

All RefSeq versions up to v98 can be included in the query by adding the flag `--all_refseq_versions`. Currently the DQT does not support user removal or addition of databases. These features are planned to be part of future releases. 

#### Details & Example

The query functionality can be run using the following command (for example):

```
python3 query_tool.py -t <taxid_source>
```

Here, `<taxid_source>` can have one of three forms:

1. A file path to a text file with a list of line-separated taxon IDs. Excluding the newline and leading or trailing whitespace, each line must readily convert to an integer or the procedure will raise an error.
2. The string `stdin` (i.e. `python3 query_tool.py -t stdin`). In this case a line-separated list (formatted as above) is expected from standard input. When an empty or all-whitespace line is encountered, input is terminated.
3. A single integer representing a taxon ID. The procedure will run for only this taxon. Additionally, the output report will have a slightly different format than for multiple IDs.

**Example**:

<details><summary>(show example)</summary>

```
(metscale) :~$ python3 query_tool.py -t testtax.txt

DB Column Names:
   1: minikraken_20171019_8GB
   2: minikraken2_v2_8GB_201904_UPDATE
   3: kaiju_db_nr_euk
   4: NCBI_nucl_wgs
   5: NCBI_nucl_gb
   6: MTSV_Oct-28-2019
   7: metaphlan3
   8: RefSeq_v98

    taxid      rank 1 2 3 4 5 6 7 8
  1251942   species - - - - 1 - - -
  1913708   species - - 1 - 1 - - -
   980453   species - - - - 1 - - -
   743653   species - - - - 1 - - -
  2196333   species - - - - 1 - - -
   146582   species - - - - 1 - - -
  1950923   species - - - 1 - - - -
  1420363   species - - - - 1 - - -
  1367599   species - - - - 1 - - -
    48959   species - - - - 1 - - -

   (...truncated...)
```

</details>


## Output
To understand how to interpret the output of the DQT we will use the example query from the previous section:

```
(metscale) :~$ python3 query_tool.py -t testtax.txt

DB Column Names:
   1: minikraken_20171019_8GB
   2: minikraken2_v2_8GB_201904_UPDATE
   3: kaiju_db_nr_euk
   4: NCBI_nucl_wgs
   5: NCBI_nucl_gb
   6: MTSV_Oct-28-2019
   7: metaphlan3
   8: RefSeq_v98

    taxid      rank 1 2 3 4 5 6 7 8
  1251942   species - - - - 1 - - -
  1913708   species - - 1 - 1 - - -
   980453   species - - - - 1 - - -
   743653   species - - - - 1 - - -
  2196333   species - - - - 1 - - -
   146582   species - - - - 1 - - -
  1950923   species - - - 1 - - - -
  1420363   species - - - - 1 - - -
  1367599   species - - - - 1 - - -
    48959   species - - - - 1 - - -

   (...truncated...)
```

For the numeric values present in the matrix there are 3 possible outcomes:
* 1: Taxon ID is present in that database
* 2: Taxon ID is not present but it's species-level ancestor is
* -: Neither is present

*Note:* For taxon IDs above species level, only outcomes 1/0 are possible.

If only a single taxon ID is input, the DQT will output the rank of that taxon ID and a `Yes` or `--` (No) response for containment in each database.
```
(metscale) :~$ python3 query_tool.py -t 10
Taxon ID:         10 (rank: genus)
DB results:
          minikraken_20171019_8GB: --
 minikraken2_v2_8GB_201904_UPDATE: Yes
                  kaiju_db_nr_euk: Yes
                    NCBI_nucl_wgs: --
                     NCBI_nucl_gb: Yes
                 MTSV_Oct-28-2019: --
                       metaphlan3: --
                       RefSeq_v98: Yes
```

## Adding a Custom Database

## Additional Information

A complete list of the commands and options is available using the `--help` flag at the command line:

```
python3 query_tool.py --help
```

### Logging Options:
Options related to how much information the program prints while running:

  `-qt`, `--quiet`      If given, disables logging to the console except for
                        warnings or errors (overrides `--debug`)
                        
  -`vqt`, `--veryquiet` If given, disables logging to the console (overrides
                        `--quiet` and `--debug`)
                        
  `--debug`             If given, enables more detailed logging useful in
                        debugging.
