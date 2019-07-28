# Australian Super insurance claim


## Architecture


![Architecture](readme/img/aus-super-ins-claim.png)

![Options](readme/img/flow.png)



The project contains the distinct phases:
1. Ingress of documents via email
2. Saving documents locally for processing
3. Classification and Extraction of data from pdf files
4. Using the extracted data for:
    * update registry
    * generate response email
5. Send email 

## Interfacing between stages


### Stages 1 - 3 
Between stages 1 and 2, as well as stages 2 and 3, the interfacing will be achieved through strings pointing to file repositories (such as SharePoint, shared drives or folders), with the the calling stage workflow passing the path to the second stage workflow

### Stages 3 and 4
Between stages 3 and 4, the interface will be an IDictionary <String, String>, passing the key-value mappings of the elements needed for the generation of the output file

### Stages 4 and 5
Stage 5 requires just the path to the local document to be attached to the email. 

## Detailed implementation

### Document sourcing
The primary document source is an email folder with rules. Accessing the attachment for local processing by the robot can be performed via 2 possible routes:
1. Use Microsoft Flow to save the attachment to Sharepoint. This can be an option if the email address is used heavily and interacting with it is very time consuming. Output of this stage: `file location in SharePoint`
2.  The robot accesses the email folder directly and downloads the attachments to a local/shared drive. Output of this stage: `file location on shared/local drive`. 
*If first and second stages are ran on different machines, attention should be paid to the location of the file and ensure that all robots have access to it*

### Downloading documents for processing
The document is copied locally into a `tmp` folder. Output of this stage: `file name in tmp folder`

### Classification and extraction

The pdf document needs to go through 3 stages before it can be presented to the next stage:
1. Loading
2. Classification
3. Extraction

#### Loading
The document is loaded using the `Read pdf` activity. The workflow is straight forward, only point of attention is around the exceptions which are thrown:
1. If the location is not reacheable
2. If the file does not exist at the location
3. If the file is of an invalid format (_ie_ not a pdf file)

It assumed the pdf file is digital, not a scanned one. The reading of the text is done natively.

#### Classification
The classification stage is performed using `RegEx`. The mapping of the regular expressions to actual suppliers is located in `src\assets\classify.csv`
In this mapping, the keys are the name of the processes the documents will start. 
The output of the classification process is an augmented pdf text which will be passed to the next stage.

#### Extraction
The classified text is received from the previous stage. Based on the clasification, a lookup is performed to find the set of regular expressions which will be used to extract the information from the text. 
The mapping of the process name to the file containing regular expressions is  `processKey` -> `src\assets\extract-processKey.csv`

The extraction workflow throws an exception if the document was wrongly classified; _ie_ if the regular expressions return a number of Matches which is different than 1, either no match or 2 or more matches

### Update registry and generate response document

#### Interfacing

The input into this stage is a key-value mapping (the implementation specific data structure used is a .NET `IDictionary<String,String>`)
The keys expected in the mapping are:
* memberName
* memberNumber
* dateSigned
* 28dayLetter
* signedOff

### Send email
The document generated at the previous stage is attached to an email and sent. the generated document is moved to the `processed` folder.
