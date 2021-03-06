<!-- PROJECT LOGO -->
<br />
<p align="center">
  <h3 align="center">UoA MyTardis Ingestion Scripts</h3>

  <p align="center">
    Ingestion scripts for the University of Auckland's customised MyTardis. For each new instrument included into MyTardis, an instrument specific parser is needed. This parser will provide metadata to the ingestion scripts, by way of an Ingestion Factory instance.
	<!--
    <br />
    <a href="https://github.com/github_username/repo_name"><strong>Explore the docs »</strong></a>
    <br />-->
    <br />
    <a href="https://github.com/github_username/repo_name/issues">Report Bug</a>
    ·
    <a href="https://github.com/github_username/repo_name/issues">Request Feature</a>
  </p>
</p>

<!-- TABLE OF CONTENTS -->
## Table of Contents

* [About the Project](#about-the-project)
  * [Built With](#built-with)
* [Usage](#usage)
* [Roadmap](#roadmap)
* [Contact](#contact)

## About the Project

[MyTardis](https://github.com/mytardis/mytardis) is an open-source scientific instrument data repository developed by Monash University. At the [University of Auckland](https://auckland.ac.nz) we have created a customised fork of the base repository to accommodate our specific researcher requirements. Part of this project involves the automated ingestion of research data into the repository and these scripts provide a standardised means of accessing the repository through the MyTardis API.

### Built With

* [Python](https://www.python.org/)

## Usage

Data ingestion into MyTardis via these ingestion scripts can be considered to have two parts, [metadata parsing](#metadata-parsing) followed by [data ingestion](#data-ingestion).

API keys, URLs for MyTardis and other information for ingestion purposes is held in **.env** files stored locally. A **pathlib** **Path** object is passed to the ingestion classes locating the config files, which are split into local and global config files, with the latter containing sensitive data beyond the scope of MyTardis (AD credentials, S3 credentials etc.) The **process_config()** function found in **config_helper.py** can be modified to account for additional API keys etc.

### Metadata Parsing

The data ingestion part of the ingestion process is required to be customised for different instruments and facility workflows. The goal of the metadata parser is to prepare the metadata into a standardised Python dictionary in preparation for ingestion. 

This process may include analysis of one or more run-file formats that are created by the instrument, which is why the parser should be customised on an instrument-by-instrument basis. The **_IngestionFactory_** class is an abstract base class for the specific parsers and the **process_object()** functions (where objects are the project/experiment/dataset and datafiles of the MyTardis hierarchy)should be defined in such a way as to extract and prepare the instrument metadata into an ingestible format.

### Data Ingestion

The data ingestion part of the ingestion process takes the prepared metadata dictionaries and calls the MyTardis API to create the objects in MyTardis. The **forge_object()** and **reforge_object()** functions allow the **_IngestionFactory_** to create the objects in MyTardis this way. Basic sanity checking is done on the input dictionaries to ensure that the minimum metadata required to create the appropriate object in MyTardis is present in the input dictionaries. We have also included functionality to mint and update RAiDs as identifiers for the different objects within MyTardis.

**Minimum metadata requirements:**

 - Project Object.
   - **name**: The project name
   - **description**: A short project description
   - **raid**: A unique project identifier, RAiD for UoA project
   - **lead_researcher**: A username for the lead researcher in the project. This user will get admin access at all levels of the project and it's child objects. It should be noted that the UoA version of MyTardis authenticates against Active Directory and the API may need reworking for OAuth authentication.
   - **schema**: A schema name as defined within MyTardis for the Project level schema. This will include the metadata fields and short names associated with them. 
   - Any additional keys in the project dictionary (with a couple of exceptions) will be added as metadata fields. If a match can be found in the schema, then this will be available for indexing for search. If not then the metadata will be added but may not be indexed.
 - Experiment Object.
   - **title**: The experiment name (*NB*: there are differences in the naming schemes between objects that may need tidying up - a legacy of the length of MyTardis development)
   - **raid**: A unique experiment identifier, RAiD for UoA experiments
   - **description**: A short description of the experiment
   - **project**: A project identifier (i.e. the **raid** field from the project object in question) for the parent project.
   - **schema**: A schema name as defined within MyTardis for the Experiment level schema. This will include the metadata fields and short names associated with them.
   - As with the project any additional keys will be added as metadata fields
 - Dataset Object.
   - **description**: The dataset name (see experiment **title** above)
   - **dataset_id**: A unique dataset identifier, RAiD for UoA datasets, could also be Dataset DOIs
   - **experiments**: A **list** of experiment identifiers associated with the experiment **raid**. We have assumed a one-to-many relationship between experiments and datasets, rather than the many-to-many relationship that is default in MyTardis. As such the scripts only get the first item in the list and would need refactoring to accommodate a many-to-many relationship.
   - **instrument_id**: A unique identifier to the instrument that the data was generated on. Currently there is no standard persistent identifier that has widespread community adoption (DOIs are the most likely candidate).
   - **schema**: A schema name as defined within MyTardis for the Dataset level schema. This will include the metadata fields and short names associated with them.
   - As with the project any additional keys will be added as metadata fields
 - Datafile Object. (*NB*: Given the limitations associated with transferring data through the *html* interface of MyTardis, we are streaming the data directly into our object store, using the **boto3** python library, and the **filehandler.py** script provides wrapper functions to do this. We then create a **replica** in MyTardis that points to the file location.
   - **filename**: The file name of the data file to be ingested
   - **md5sum**: The MD5 checksum of the original data file
   - **storage_box**: The MyTardis storage box defined for the facility
   - **local_path**: The full path to the local instance of the data file to be ingested
   _ **remote_path**: The relative path to the remote instance of the data file for the purposes of maintaining the local directory structure. This is in place to accommodate analysis packages that expect a specific directory structure.
   - **full_path**: The full path to the remote instance of the data file (normally constructed from the **remote_path** by the parser.
   - **schema**: A schema name as defined within MyTardis for the Datafile level schema. This will include the metadata fields and short names associated with them.
   - As with the project any additional keys will be added as metadata fields
   
### Roadmap

TBD

## Contact

Contact Chris Seal (c.seal@auckland.ac.nz) for more information.
