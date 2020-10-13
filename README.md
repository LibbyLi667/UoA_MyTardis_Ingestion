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
* [License](#license)
* [Contact](#contact)

## About the Project

[MyTardis](https://github.com/mytardis/mytardis) is an open-source scientific instrument data repository developed by Monash University. At the [University of Auckland](https://auckland.ac.nz) we have created a customised fork of the base repository to accommodate our specific researcher requirements. Part of this project involves the automated ingestion of research data into the repository and these scripts provide a standardised means of accessing the repository through the MyTardis API.

### Built With

* [Python](https://www.python.org/)

## Usage

Data ingestion into MyTardis via these ingestion scripts can be considered to have two parts, [metadata parsing](#metadata-parsing) followed by [data ingestion](#data-ingestion).

### Metadata Parsing

The data ingestion part of the ingestion process is required to be customised for different instruments and facility workflows. The goal of the metadata parser is to prepare the metadata into a standardised Python dictionary in preparation for ingestion. 

This process may include analysis of one or more run-file formats that are created by the instrument, which is why the parser should be customised on an instrument-by-instrument basis. The **_IngestionFactory_** class is an abstract base class for the specific parsers and the **process_object()** functions (where objects are the project/experiment/dataset and datafiles of the MyTardis hierarchy)should be defined in such a way as to extract and prepare the instrument metadata into an ingestible format.

### Data Ingestion

The data ingestion part of the ingestion process takes the prepared metadata dictionaries and calls the MyTardis API to create the objects in MyTardis. The **forge_object()** and **reforge_object()** functions allow the **_IngestionFactory_** to create the objects in MyTardis this way. Basic sanity checking is done on the input dictionaries to ensure that the minimum metadata required to create the appropriate object in MyTardis is present in the input dictionaries. We have also included functionality to mint and update RAiDs as identifiers for the different objects within MyTardis.

**Minimum metadata requirements:**

 - Project Object.
 
 ..- 'name': The project name
 
 ..- 'description': A short project description
 
 ..- 'raid': A unique project identifier, RAiD for UoA project
 
 ..- 'lead_researcher': A username for the lead researcher in the project. This user will get admin access at all levels of the project and it's child objects. It should be noted that the UoA version of MyTardis authenticates against Active Directory and the API may need reworking for OAuth authentication.
 
 ..- 'schema': A schema name as defined within MyTardis for the Project level schema. This will include the metadata fields and short names associated with them. 
 
 ..- Any additional keys in the project dictionary (with a couple of exceptions) will be added as metadata fields. If a match can be found in the schema, then this will be available for indexing for search. If not then the metadata will be added but may not be indexed.
 - Experiment Object.
 - Dataset Object.
