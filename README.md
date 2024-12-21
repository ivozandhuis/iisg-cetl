# CETL

*NEEDS UPDATING AFTER REFACTORING*

Principles: 
* All scripts are independent. You could run them in random order. Sometimes because the transformation script changed, sometimes because the data changed, sometimes both.
* If a script aborts unexpectately, the chain should not be broken. Calling it a new, makes it pick up were it left off.
* be generous with data storage ('cache') so you could study all steps
* NOT: performance
* NOT: on-the-fly synchronization

These scripts: 

0. Monitor which records are created, updated or deleted.
1. Extract metadata from a collection management system.
2. Transform data into RDF
3. Load the result into an appropriate databasesystem

For instance: for biblio it Monitors via the ListIdentifiers verb of OAI-PMH which bibliographical records are changed in Evergreen, Extracts the metadata with a GetRecord request on the OAI-PMH endpoint, Transforms it into RDF (TBD: test the BIBFRAME transformations of the LoC) and finally uploads the data into TriplyDB.

Other example: for ORCID a list of all the relevant ORCIDs is manually composed, Extracts the data via content negotiation on orcid.org, (does not need to transform, because it is already RDF) and creates a zip-file with the records that can be uploaded into Druid (TBD: access Druid through the API)

## Pattern
The pipeline is managed through a database 'status.db' with identifiers needed to download the records identified with these identifiers. The check-step finds the newly created, updated or deleted identifiers, stores the status of the processing and the timestamps of the moments the record is checked, extracted, transformed and loaded.

## Prerequisites
The scripts run in Python3 and need the following libraries:
- sqlite3
- os
- datetime
- pathlib
- csv
- json
- requests
- Sickle (OAI-PMH library)

Other git repo's with code needed to run the cetls:
- rico-converter
- marc2bibframe2

## Check
The check script creates and updates the list of identifiers to be downloaded, for instance with the OAI-PMH verb ListIdentifiers. 

todo:
* how to update this list

## Extract
With the identifiers in the status.db we download the records record by record, for instance with the OAI-PMH verb GetRecord or content negotiation. 

The downloaded data is stored in files, a file for every record, named with the number of the identifier. Because the number of records could be huge (eg 1.2 million for the bibliographical data) we store them in a balanced directory structure based on the last three digits of the (numerical) identifier.

Run the extract command from the specific directory (e.g. archives):

```python extract.py```

todo:
* how to incrementally update the data (instead of downloading everything everytime)

## Transform
For the transformer you will need the appriote xslt stylesheet. E.g. for archives we use [Ivo's ead2rico](https://github.com/ivozandhuis/ead2rico).
Run the extra command from the specific directory providing the stylesheet as command line argument:

```python transformer.py /Users/me/git/ead2rico/xsl/ead2rico.xsl```

## Load
TBD

