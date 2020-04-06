# Template document manipulation project

[![Build Status](https://dev.azure.com/andreibarbu0946/UiPathGoodPractices/_apis/build/status/AndreiBarbuOz.uipath-doc-manipulation?branchName=master)](https://dev.azure.com/andreibarbu0946/UiPathGoodPractices/_build/latest?definitionId=12&branchName=master)

## Purpose

This template provides a template of UiPath workflows, creating a framework for a common scenario. The purposes of the project are to:
1. Break down the process into stages with clear interfaces between them (ingress, classify-extract, process, outgress)
2. Showcase multiple ways of processing the same item (single loop, multiple loops, ReFramework, ReFramework light with Excel input)
3. Provide examples of multiple interchangeable workflows implementing the same stage (ingress via Email, or Sharepoint, or Google docs), all of them adhering to the same interface convention
4. Exemplify complex data structures which allow for easily extensible implementations (dictionaries, collections, tuples)
5. Implement an example of good practices, limiting the number of arguments, commenting the code, handling exceptions

## Scenario
A sample project showcasing the UiPath implementation of a common scenario: 
1. Agents fill client information into template excel files and post these files using one of a few possible ways (email to a specified address, sharepoint, google docs)
2. The files are downloaded locally for classification, data extraction and processing
3. The excel files are opened and relevant data is extracted
4. Some of the fields are used to fill in a template document
5. The document is saved as a pdf and sent via email to an email address provided in the input excel file

## Architecture

![Architecture](https://www.lucidchart.com/publicSegments/view/76e2eb65-d3f9-414b-a2ee-49053600a1aa/image.png)


The project contains four distinct phases:
1. Ingress of documents, saving them locally for further processing. Currently, only email is implemented
2. Extraction of data from the input files, using the folder where the files were saved as source
3. Using the extracted data to fill in templates and save them in the local outgress folder
4. Ougress documents using one possible option. Currently, only email has been implemented


## Installation and configuration

Clone the repository and update the config file to be used. Configuration file needs to be updated with required keys. Among them, the outlook folders config and local temporary paths. Sample input files can be found in the `tests\assets` folder.

## Implementation scenarios

### Single loop
![Single Loop](https://www.lucidchart.com/publicSegments/view/546caffb-a502-4a5f-a9eb-f94693813805/image.png)

The single loop approach creates a single loop, which takes the attachments from the ingress email, downloads them, processes and then outgresses the outcome

### Multiple loops
![Multi Loop](https://www.lucidchart.com/publicSegments/view/ed3d773c-e6b6-4b62-9e2b-b35c79425f6b/image.png)

The multiple loops approach creates loops for each of the stages and performs the actions for all the transactions before moving to the next stage 

### REFramework
The RE Framework uses a single loop approach with an added layer of exception handling and transaction queueing.

## Interfacing between stages

### Stages 1 - 2 
Between stages 1 and 2, the interfacing will be achieved through a common folder where files being ingressed will be saved for further processing. Optionally, the path to the document can be added to a transactions queue on Orchestrator, for logging and monitoring purposes.

### Stages 2 and 3

Between stages 2 and 3, the data will flow using a Collection of `IDictionary <String, String>`, passing the key-value mappings of the elements needed for filling the template and the generation of the output file

The keys expected in the mapping are:
* firstName
* lastName
* address
* custId
* account
* bsb

### Stages 3 and 4

Stage 4 requires just the path to the local document which is being outgressed

## Detailed implementation

### Document ingress

The primary document source can be one of multiple options, such as an email folder or a sharepoint folder. The document will be downloaded for processing, to a local folder. Downloading the file can take place via these means:
1. Save the the file from the Sharepoint location provided as an argument.
2. The robot accesses the email folder directly and downloads the attachments to a local/shared drive.  
*If these ingress routes can be run on on different machines, attention should be paid to the location of the file and ensure that all robots have access to it*

The document is copied locally into a temporary folder, to be processed. Output of this stage (which is optional): `file name in tmp folder`

### Data extraction and manipulation

In this stage, the data is extracted from the ingress `.xlsx` file. The format of the file is considered known, a template filled by the sender.

Data validation is performed in this stage, validating inputs such as account and BSB formats

The document is protected with a password. The documents contained in the assets folders of the project have the password `password`

### Data usage for filling in the template document 

Data is received in the key-value mapping and is used for filling in the required fields in the outgress/template document. The `.docx` document is first filled and then printed as a pdf file, ready to be sent to its destination. The processed documents are moved from the `processing` folder to its final destination folder `processed`

### Outcome and document outgress
The document generated at the previous stage is attached to an email and sent. The document is then moved to the `sent` folder.


## Testing and development

For most of the files in the `process\src` folder, an equivalent test file should exist in the `process\tests` folder. The functionality of the workflow is embedded in the tests. _ie._ the successful execution of the tests define the correct behavior of the workflow.

## CI/CD

![CI/CD](https://www.lucidchart.com/publicSegments/view/96f2f529-1e97-4cb7-8793-1a70c0e82de3/image.png)
