# Treatment checker integration guide

Here, you'll find a documentation and clear instructions for integrating your applications with Apixmed genetic interpretation module.

For whom
This guide is intended for individuals responsible for managing interactions between their system and the Apixmed system. 
It also serves as an official document describing the provided services from a legal perspective.

How It Works

Merchant interacts with the application through shared data storage (for now SFTP will be used). 
The application monitors shared data storage and when the new data (patient information and genetic data) appears, it uploads that data to the internal server.
The application processes and analyzes provided data, and then returns the result back to the shared data storage. 
Merchants can download and use the processed results for their purposes.
This process is automated, and the interaction protocol with the application is described in detail below.

> [IMPORTANT]
> 
> First, you need to set up your organization ID and choose an organization account for your company.
> 
> Unfortunately, there is no sandbox you can use without interacting with our tech team.
> Our team will be responsible for setting up this system manually. Kindly prepare the following information and share it with us and send it via email on support@apixmed.com. 

## 1. Organization settings
 
 
Organization is key information that will be used as an aggregator for invoicing for used service. The presence of an Organization data only leads to the creation of metadata about the client and does not result in the creation of technical infrastructure.  

- **Organization**
  - **Id** - (*required*) an alphanumeric string (English letters and numbers); this Id will be used in Apixmed API and front-end URLs.
  - **Email** - (*required*) a contact email of a representative of the organization, intended for notification and changes related to organization.

Organization Account is a record of information to create a dedicated storage for exchanging incoming data (genetic files) and outgoing data (the results of genetic file analysis). An organization may have multiple accounts, which results in multiple data storages with different pricing plans and behavior.

- **Account**
  - **Id** - (*required*) an alphanumeric string (English letters and numbers); this Id will be used in Apixmed API URLs.
  - **Name** - (*required*) the user-friendly name of the account that will be used for internal and cross-team usage (final users will not be able to see this).
  - **Description ** - (*required*) a concise description of your organization, also intended for user display on the Apixmed platform (recommended to not exceed 100 words or 500 characters).
  - **Email ** - (*required*) a contact email of a representative of organization, intended for notification and changes related to the organization account. 

<details>
  <summary>Example</summary>

- **Organization**
  - **Id** = `"test"`
  - **Email** = `"org-contact-email@domain.com"`
 
- **Account**
  - **Id** = `"org-test-pharmacogenomics-only"`
  - **Name** = `"Test Organization Account Pharmacogenomics"`
  - **Description** = `""This organization account intended for analysis related to pharmacogenomics"`
  - **Email** = `"org-contact-email@domain.com"`

</details>

## 2. File storage

All data is stored in SFTP storage and accessible from the Apixmed team and merchant team. 
The credentials are: SFTP host, username and password OR private key file which can be taken from Apixmed team.

**Required data:**
- **Sftp host** - domain.blob.core.windows.net
- **Username** - OrgUserName
- **Password** - P@ssword

or

**Required data:**
- **Sftp host** - domain.blob.core.windows.net
- **RSA PRIVATE KEY** - file without extension with described RSA PRIVATE KEY  

File storage folder structure.

SFTP storage has a next folder structure
- archive
  - 1.1 genetec-files
  - 1.2 orders
- inbound 
  - 2.1 genetec-files
  - 2.2 orders
- outbound
  - 3.1 pdf-reports
  - 3.2 23andme-files
  - 3.3 html-reports
- samples

### Archive

This folder is used to store historical data. The contents of this folder and its subfolders consist of files that remain unchanged by the system and must not be deleted. This data serves as a historical record.

### Archive/genetic-files

This subfolder is used to store user input of genetic files as a historical record. After any result of analysis raw

### Archive/orders

This subfolder is used to store user input of order files as a historical record. After any result of processing orders, the orders file will be moved here with a corresponding subdirectory.

### Inbound

This folder serves as the entry point for the system to retrieve user input for further processing. Merchants must upload all input files exclusively to this location and its subdirectories.

### Inbound/genetic-files

Merchants should upload raw genetic data files to this subfolder. Each upload can be placed in a shared subdirectory with any name - this subdirectory will then be used by the system to generate output.

- Supported raw genetic file formats:
  - .vcf
  - .DNARawData.txt (Muhdo, the filename should end with '.DNARawData' so that the algorithm can correctly identify the actual data for proper handling)
  - .txt (Eurofins Final Report) 

It must be uploaded before the related ‘order file’ is uploaded.

### Inbound/orders

This subfolder is intended for merchants to upload files containing data for incoming orders. An order represents all the necessary information required to generate a genetic report. Files can have any name and be placed in any subdirectory within this folder. The only requirement is that the file has a valid extension. After handing the orders file it is moved to /archive/orders. 

Each row in the file must reference a specific raw genetic file located in /inbound/genetic-files. This file may be either for an individual or part of a batch containing multiple patients. If referencing a batch file, the order must include the patient identifier as specified in that batch file. The orders file acts as the entry point for genetic analysis. 

It must be uploaded after raw genetic data files.

- Supported types for order file:
  - .csv

- **PatientId** - (required) is a string identifier for a patient that was analyzed by the laboratory. It must be defined in the raw genetic file to find all needed alleles for the patient in case the raw genetic file has multiple patient data.
- **FullName** - (required) is a string display name which is used in generated reports. Can be any input 
- **Age** - (required) is a string display age which is used in generated reports. Can be any text with length from 1 to 3 symbols.
- **Sex** - (required) is an enumeration which represents the biological sex of the patient. 
  - **Values**:
    - “M” - Male
    - “F” - Female
    - “U” - Undefined
- **ReportCategory** - (required) is a category of report
  - **Values**:
    - Wellness
    - Disease
- **ReportType** - (required) is a type of report
  - **Values**:
    - Wellness
	- Beauty
	- Sport
    - Diet
	- Pharmacogenomics
	- HeartDiseases
    - Cancer 
	
- **Language** - (required) is a language which is used for report generation
  - **Values**:
    - Ukrainian
	- Polish
	- English
- **ProcessType** - (required) is used to determine whether the order is new or needs to be recreated because it was previously processed.
  - **Values**:
    - Initial
    - Reprocess

> [IMPORTANT]
> Incorrectly filled data may result in inaccurate generation or prevent the generation process entirely.

### Outbound

This folder is used to store the system's output, including processed orders and their related genetic files. 

### Outbound/23andme-files

This subfolder stores 23andMe raw genetic data files for processed orders.
Processed files are stored in the corresponding input subdirectory within a directory named with the current date in the format YYYY-MM.

### Outbound/html-reports

This subfolder stores html raw genetic reports for processed orders.
Processed files are stored in the corresponding input subdirectory within a directory named with the current date in the format YYYY-MM.

### Outbound/pdf-reports

This subfolder stores PDF genetic reports for processed orders. All reports are generated using samples from the /samples folder. Any modifications or suggested changes should be based on the provided samples.
For repeated report generation -whether it's a new report or a recreation within the same month - there may be multiple files with the same name in the current month's folder.  To preserve all versions of files, the old file will be moved to the  {filename}-old{sequenceNumber}  folder, and the new file will be added to the root. The most up-to-date versions of files are always stored in the root folder.
 
Processed files are stored in the corresponding input subdirectory within a directory named with the current date in the format YYYY-MM.

### Samples

This subfolder contains the current samples used to generate PDF reports. Samples are organized in a subdirectory named by date, and include examples for all report types.

