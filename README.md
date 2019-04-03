# Prior Art Archive Overview
Low quality patents waste money. Fewer bad patents will be issued if we as industry give USPTO examiners the tools they need to find old technology. 

US companies spend millions of dollars each year defending against patents that shouldn’t be issued. The patent examination process should stop patents from being issued on old or obvious technology.  Unfortunately, just because technology is old doesn’t mean it is easy for patent examiners to find. Particularly in computer software and hardware, much prior art is in the form of old manuals, documentation, web sites, etc. that have, until now, not been readily searchable.

MIT and Cisco have collaborated to create this first free and open archiving platform for the entire IT industry.

## Features and Scope

* A customized parser designed to USPTO needs, with no additional training for USPTO examiners. They can use their existing set of operators and search strings to search content.
* Uploaded documents are machine-classified with CPCs by Google https://publicpolicy.googleblog.com/2015/07/good-patents-support-innovation-while.html
* Auto Save Search queries and Custom reporting to be used by USPTO to attach to their reports to support their decision
* Multi-tenant open architecture. You maintain your own content through your userid 
* Backend API Support & an open standard lets USPTO or any other company develop new search tools, using machine learning or other tools to re-use content, improve review and research during patent filing. 
* Hosted by MIT and open to the world 

## Benefits
The platform is being developed with the USPTO, and available to patent examiners.
The documents will be indexed by Google Patents and Bing, and made available to search engines for easy access to the public.
Applicants can search for prior art before they file patent applications.
The site can be used by innovators and law firms to review prior innovations and documents before they file a patent request.

## How it Works
If you have questions or comment, you can reach us at priorartarchive@media.mit.edu

Prior art files are associated with the uploading account.  You can create an account at https://www.priorartarchive.org/signup.

Users can connect to our secure FTP server with their accounts.  FTP details:  
 Host: sftp.priorartarchive.org
 Port: 22
 Protocol:  SFTP
 Username: <your username>
 Password: <your password>

Once connected, users can upload prior art through their FTP client. It is then added to the search index, and once a week is automatically given approximate CPC classifications by a parser.  Content will be made available to other Google Patents & Bing so that it is findable via public search engines.

* Documents are generally tagged and added to the search index within minutes.  Supported filetypes include Word, pdf, and text documents, Excel spreadsheets, images, web pages, and video).   
* Make sure your documents include the following metadata elements where possible:
 _Title, Description, Creation Date, Publication Date, Modification Date, Copyright_


# Architecture
The following diagram depicts the architecture for the Prior Art Archive and all associated services. The blocks are described below:

![Prior Art Archive architecture diagram](https://i.imgur.com/1mGeoFZ.png)

- **FTP client:** A user's personal FTP client. They should login using the username and password they setup when they signed up.
- **priorart-sftp-authentication:** A lambda trigger that validates against the user database to authenticate the secure FTP session. [Code available here](https://github.com/knowledgefutures/priorart-sftp-authentication).
- **SFTP Server:** A hosted AWS Transfer SFTP server. Users are jailed to their home directory and cannot see files uploaded by other users. The SFTP server is a means of uploading, not file management. Removing files from the SFTP server does not delete them from the Prior Art Archive.
- **priorart-sftp-copy:** A lambda trigger that adds metadata and copies the SFTP-uploaded files to the primary file storage. [Code available here](https://github.com/knowledgefutures/priorart-sftp-copy).
- **Primary File Storage:** An S3 bucket that hosts all the content uploaded by users.
- **priorart-site:** The main repo for the Prior Art Archive project. [Code available here](https://github.com/knowledgefutures/priorart-site). This repo implements the client facing site as well as the required backend. API endpoints that talk to the Prior Art Archive Postgres DB are implemented here.
	- **User authentication:** An API route that authenticates users for SFTP and site login.
	- **Drag-and-drop upload:** An interface that allows users to directly upload files through the site.
	- **Sitemap generator:** An interface that produces a sitemap of all assets. Used by Google to generate CPC codes.
	- **Search input:** An endpoint that takes a users string input.
	- **Search results:** An interface that presents a list of search results.
- **priorart-search-parser:** A lambda trigger that takes the string input from a user and parses it into an ElasticSearch query object.
- **ElasticSearch:** An elastic.co hosted ElasticSearch cluster.
- **priorart-file-parser:** A server deployment on Elastic Beanstalk that implements Tika parsing, formats metadata as Underlay assertions, and writes content to the Postgres DB, Elasticsearch, and the Underlay. [Code available here](https://github.com/knowledgefutures/priorart-file-parser).
	- **tika:** Tika server that extracts metadata from a file.
	- **underlay formatter:** Parses metadata into JSON-LD compliant Underlay assertions.
- **Underlay cluster:** The Underlay cluster that serves as ground truth for all data in the Prior Art Archive. [Details about the Underlay cluster can be found here](https://kfg.mit.edu/pub/l18rh143).
- **Postgres DB:** A hosted Postgres DB that serves user and document information for the user-facing site.
- **Google CPC Generation:** A black box run by Google that generates CPC codes from files.
- **CPC Ingestion:** A server that ingests CPC code metadata from Google and writes it to the Underlay cluster, Elasticsearch, and the Postgres DB.

