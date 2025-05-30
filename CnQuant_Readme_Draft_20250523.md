# **Table of contents**

* [Disclamer](https://github.com/neuropathbasel-pub/CnQuant#disclamer)  
* [Introduction to the app](https://github.com/neuropathbasel-pub/CnQuant#introduction-to-the-app)  
* [Requirements](https://github.com/neuropathbasel-pub/CnQuant#requirements)  
  * [Directory structure and data annotation files](https://github.com/neuropathbasel-pub/CnQuant#directory-structure-and-data-annotation-files)  
  * [Software requirements](https://github.com/neuropathbasel-pub/CnQuant#software-requirements)  
  * [Manifests, reference IDAT pair, and data annotations download](https://github.com/neuropathbasel-pub/CnQuant#manifests-reference-IDAT-pair-and-data-annotations-download)  
* [Execution with CQmanager](https://github.com/neuropathbasel-pub/CnQuant#execution-with-cqmanager)  
* [Running CnQuant apps in WSL2](https://github.com/neuropathbasel-pub/CnQuant#running-CnQuant-apps-in-wsl2)  
  * [WSL2 image preparation](https://github.com/neuropathbasel-pub/CnQuant#wsl2-image-preparation)  
  * [CN analysis with CQcalc](https://github.com/neuropathbasel-pub/CnQuant#cn-analysis-with-cqcalc)  
  * [CN Summary Plots with CQall\_plotter](https://github.com/neuropathbasel-pub/CnQuant#cn-summary-plots-with-cqall_plotter)  
  * [Per-case results inspection with CQcase](https://github.com/neuropathbasel-pub/CnQuant#per-case-results-inspection-with-cqcase)  
  * [CN Summary Plots inspection with CQall](https://github.com/neuropathbasel-pub/CnQuant#cn-summary-plots-inspection-with-cqall)  
* [Alternative ways to install and run CnQuant apps](https://github.com/neuropathbasel-pub/CnQuant#alternative-ways-to-install-and-run-cnquant-apps)

# **Disclaimer**

Implementation of this software in a diagnostic setting occurs in the sole responsibility of the treating physician. Usage of this software occurs at the risk of the user. The authors may not be held liable for any damage (including hardware) this software might cause. Use is explicitly restricted to academic and notn-for-profit organizations.

# **Introduction**

CnQuant, is a lightweight stack of 5 applications designed to streamline the analysis of Illumina Infinium Methylation array data through a web browser CnQuant consists of the following programs (stored in separate repositories) for CNV calculation and visualization:

* [CQmanager](https://github.com/neuropathbasel-pub/CQmanager)  
* [CQcalc](https://github.com/neuropathbasel-pub/CQcalc)  
* [CQall plotter](https://github.com/neuropathbasel-pub/CQall_plotter)  
* [CQcase](https://github.com/neuropathbasel-pub/CQcase)  
* [CQall](https://github.com/neuropathbasel-pub/CQall)

CnQuant seamlessly unifies integrates raw data from multiple kit versions (450k, EPICv1, and EPICv2) by , merging it into an Illumina Infinium Methylation array 450k format for consistent downstream processing. Based on Leveraging this the harmonized data, [CQcalc](https://github.com/neuropathbasel-pub/CQcalc) computes copy number variants (CNVs) and [CQcase](https://github.com/neuropathbasel-pub/CQcase) visualizes these results for every processed IDAT pair, displaying potentially actionable targets in point-of-care settings. offering clinicians clear, actionable informationinsights. Beyond case-specificbased CNV analysis, the [CQall plotter](https://github.com/neuropathbasel-pub/CQall_plotter) computes generates tumor entity-specific summary plots from case collections, which that highlights recurrent CNV patterns. SThe summary plots are visualized with [CQall](https://github.com/neuropathbasel-pub/CQall). In sum, CnQuant simplifies complex workflows, delivering robust results in short time in an intuitive package. CnQuant is optimized for low-latency data visualization and inspection. The CNV Microarray data areis pre-analyzed on a local server and stored in p[Parquet](https://parquet.apache.org/) files for rapid viewing and examination through the web-based frontends [CQcase](https://github.com/neuropathbasel-pub/CQcase) and [CQall](https://github.com/neuropathbasel-pub/CQall).

# **Requirements**

A If you do not have Docker installed on your system, proceed to Docker installation is required. documentation for your platform.  
After the installation, eEnsure to carry out the Docker post-installation steps to run the container as a non-root user. If new to docker, the If you are not yet familiar with docker, you might be interested in [Docker Curriculum](https://docker-curriculum.com/). might be helpful.

In case of running CnQuant applications on a server without internet access, you will need to download all docker images elsewhere and transfer them to your Docker host machine:

docker pull \\  
neuropathologiebasel/cqcalc:latest \\  
neuropathologiebasel/cqcase:latest \\  
neuropathologiebasel/cqall:latest \\  
neuropathologiebasel/cqall\_plotter:latest

docker save \-o cnquant\_images.tar \\  
neuropathologiebasel/cqcalc:latest \\  
neuropathologiebasel/cqcase:latest \\  
neuropathologiebasel/cqall:latest \\  
neuropathologiebasel/cqall\_plotter:latest

\# Before this command, the cnquant\_images.tar needs to be transferred  
\# to the host machine without internet connection

docker load \-i cnquant\_images.tar

A Docker alternative is Another option to run the CnQuant apps is WSL2 (Windows subsystem for Linux 2)Windows, described in [Running CnQuant apps in WSL2](https://github.com/neuropathbasel-pub/CnQuant#running-cnquant-apps-in-wsl2).

Primarily intended for developers or embedded systems with limited resources or CPUs other than x86\_64, all CnQuant components can be installed from source with pip. Note that CnQuant depends on both Python and R, requiring R-4.3.0 in addition to an appropriate Python environment. Installation instructions are described separately in each application component repository., The last option (not recommended) is to install the applications with pip but it will require R-4.3.0 installation with CnQuant R dependencies, which is described separately in each of the applications repository.

## **Directory structure and data annotation files**

For convenience, the same .env file is used for each app. The .env file needs to be located in each apps' main directory or it's parent directory. The .env file in an application directory has precedence before the .env file in the parent's directory. An "example.env" file is provided in each application's directory or in the parent's directory. Adapting Modifying .env files for WSL2 is typically not necessary.

## **Environment variables**

Before running any of the 5 applications (not needed for WSL2), you will need to create the following directories and define them in your .env file. Download [example.env](https://github.com/neuropathbasel-pub/CnQuant/blob/main/example.env) into your environment directory or in it's partent directory where you wish to install one of the CnQuant apps, rename it to .env and adjust the paths inside the .env file.

Note

On some operating systems, Some visual file explorers (Thunar, Gnome, Explorer) do not display files starting with a "." in their filename by default and might require settings adjustment..  
Refer to your system's manual on how to display .env if need be.

Local directories

* log\_directory: log files in plain text format will be saved here  
* idat\_directory: 'this directory contains raw IDAT file pairs (\*\_Red.idat and \*\_Grn-idat). Subfolders will not recursively be checked for IDAT pairs.  
* reference\_directory: directory, where preprocessed reference methylation data will be stored here in a binary format.  
* diagnoses\_directory: directory which has to contain two comma-separated value (CSV) files: data\_annotation.csv with two headers, Sentrix\_id and MC. data\_annotation.csv, which will contains annotated Sentrix IDs and their user-defined methylation classes. The file reference\_data\_annotation.csv has the same structure. with Sentrix IDs annotated as "reference"  
* cn\_results\_directory: directory where CNV results will be stored here in compressed Pparquet format.  
* summary\_plots\_base\_directory: summary plots will be stored here.  
* temp\_directory: this directory will be used to store temporary files, which, during normal operation, should be automatically removed if no longer required.  
* manifests\_directory: this directory will contains Illumina manifest filess for array conversion and analysis

Remote server directories

The following remote directories are required for running CQcase and CQall on a remote server. CQmanager launches runs CQcalc on the same server where you run CQmanager. CQmanager does not sync data. A The remote server could be a resource-limited system in a DMZ to provide public or institutional web interface access (CQcase and CQall), while the remaining other CnQuant are background applications intended to run that would likely would probably run on a system with larger resources.

* remote\_server\_log\_directory: log files from apps running on a remote server via CQmanager in plain text form will be saved here.  
* remote\_server\_idat\_directory: this directory on a remote server must contains raw IDAT file pairs (\*\_red.idat Red and \*\_grn.idatGreen). No recursive subdirectory checks occur. Subfolders will not be checked for existing IDATs. The directory is intended fFor apps running on a remote server via CQmanager, e.g., for files coming from an IDAT upload utility like the one in [epidip.org](http://epidip.org)..  
* remote\_server\_reference\_directory: directory on a remote server, where preprocessed reference methylation data are will be stored in binary format. For apps running on a remote server via CQmanager.  
* remote\_server\_diagnoses\_directory: directory on a remote server which has to contain two csv files: data\_annotation.csv with two headers Sentrix\_id and MC, which will contain annotated Sentrix IDs and their annotated tumor types. The second file, reference\_data\_annotation.csv, has the same structure with Sentrix IDs annotated as "reference".  
* remote\_server\_cn\_results\_directory: directory on remote directory where CNV results will be stored in compressed parquet format. For CQcase running on a remote server via CQmanager.  
* remote\_server\_summary\_plots\_base\_directory: summary plots will be stored here on the remote server. For CQall running on a remote server via CQmanager.

CQcalc-specific settings

* process\_inconvertible\_sentrix\_ids: set this to True to allow reprocessing Sentrix IDs that were previously determined to be corrupt. Default is False. Note: Depending on the amount of IDAT files present on a system, reprocessing many files can require significant amounts of time and resources.  
* check\_if\_idats\_have\_equal\_size: set this to True to reject IDAT pairs with unequal file size. Default is True. Note: Valid IDAT files are typically in the same size range.  
* minimum\_idat\_size: set here minimum idat size in MB per file. Default is 1\. Note: For more stringent filtering (many IDAT file types are larger than 1 MB) increase according to expected file size.

CQall\_plotter-specific settings

* minimal\_number\_of\_sentrix\_ids\_for\_summary\_plot: controls minimum number of analyzed samples per summary plot. Default is 5\.

Server name displayed in CQcase and CQall

* server\_name: A string or sentence wrapped in double quotes you put here will be displayed in bottom right corner in CQcase and CQall

CQcase- and CQall-specific settings

* cqcase\_host\_app\_port: port where you will be able to access CQcase. Default is 8052  
* cqall\_host\_app\_port: port where you will be able to access CQall. Default is 8050  
* email\_notification\_port: Port for SMTP server for email notifications. Default is 587  
* workers: Number of workers for [Gunicorn](https://gunicorn.org/) for running CQcase and CQall. Default is 10, suitable for multiprocessor systems. If you are running the apps on a desktop PC for a single user only, set it to 1\.  
* timeout: Timeout for caching. Default is 300\. Unit is seconds.  
* REDIS\_HOST=localhost: host for [Redis](https://redis.io/) caching. Default is localhost  
* REDIS\_PORT: Port for Redis caching. Default is 6379\.  
* maximum\_number\_of\_genes\_to\_plot: Maximum number of additional genes plotted in CQcase and CQall. Default is 600\. Note that plotting too many genes will make the plot quite unreadable and will slow down the application.  
* CACHING\_DB\_cqcase: Redis caching database number for CQcase. Default is.  
* CACHING\_DB\_cqall: Redis caching database number for CQall. Default is 1\.

Docker images

* cqcalc\_image: CQcalc Docker image name. Default is neuropathologiebasel/cqcalc:latest  
* cqcase\_image: CQcase Docker image name. Default is neuropathologiebasel/cqcase:latest  
* cqall\_image: CQall Docker image name. Default is neuropathologiebasel/cqall:latest  
* cqall\_plotter\_image: CQall plotter Docker image name. Default is neuropathologiebasel/cqall\_plotter:latest

CQmanager-specific settings. To control containerized CQviewers on remote server, setting up ssh keys is needed

* CQmanager\_gunicorn\_host\_address: Address where to run CQmanager. Default is 127.0.0.1.  
* CQmanager\_gunicorn\_port: Port on which to run CQmanager. Default is 8002\.  
* cnv\_register\_file\_name: File name for a register of processed Sentrix IDs. Default is cnv\_conversion\_register.csv.  
* CQ\_manager\_batch\_size: Number of Sentrix IDs per CQcalc container. Default is 100\. Due to R memory leaks, it is recommended to tune this parameter moderately. Depends on the available RAM and CPU threads.  
* CQ\_manager\_batch\_timeout: Seconds after CQmanager starts, a container that did not reach set batch size. Default is 300\.  
* max\_number\_of\_cqcalc\_containers: Maximum number of CQcalc containers running in parallel. Default is 10\. Depends on your available RAM and CPUs.  
* run\_CQviewers\_on\_remote\_server: Set it to True if you wish to run CQcase and CQall on a remote server. Default is False. Remember that you will need to set up a ssh key and provide username and host. Docker needs to be installed on the host.  
* CQviewers\_host: Your remote server host name or ip.  
* CQviewers\_user: Your remote server username. This user needs to be able to run Docker containers.  
* CQall\_container\_name: Name for CQall container for Docker naming purposes. Default is cqall.  
* CQcase\_container\_name: Name for CQcase container for Docker naming purposes. Default is cqcase.  
* cnquant\_redis\_name: Name for Redis container for Docker naming purposes. Default is cnquant\_redis.  
* CQviewers\_docker\_network\_name: Name for the network that will be created for CQcase and CQall. Default is cnquant\_network.  
* initiate\_cqcase\_and\_cqall\_on\_startup: Set it to True if you wish to start CQcase and CQall with Redis containers on CQmanager startup. Default is False.  
* remote\_user\_id: remote user ID for executing containers with. Required for file permissions.  
* remote\_group\_id: remote group ID for executing containers with. Required for file permissions.

Email notification settings

* crash\_email\_sender: sender email address for notification emails in case of a crash. Works with Gmail.  
* crash\_email\_receivers: Comma-separated list of email recipient email addresses for crash notifications.  
* crash\_email\_sender\_password: Application password for sender's email address. Wrap it in double quotes.

Data annotation URLs

* DATA\_ANNOTATION\_SHEET: URL for google sheet csv output for annotations  
* REFERENCE\_DATA\_ANNOTATION\_SHEET: URL for google sheet csv output for reference annotations  
* URL Example: [https://docs.google.com/spreadsheets/d/e/2PACX-1vRhQ7Cr3aBo8W9Ne8DAehMvFRxYd395ENIW9giK2ATQ3QSrM8jA2E7xXbnW7CWKMdh0IhN0YqWn37Wr/pub?gid=0\&single=true\&output=csv](https://docs.google.com/spreadsheets/d/e/2PACX-1vRhQ7Cr3aBo8W9Ne8DAehMvFRxYd395ENIW9giK2ATQ3QSrM8jA2E7xXbnW7CWKMdh0IhN0YqWn37Wr/pub?gid=0&single=true&output=csv)

## **Software requirements**

* Ubuntu 20.04 or 22.04 (other distributions have not been tested)  
* Python 3.10, 3.11, or 3.12  
* Docker (recommended) or WLS2 (CQmanager is not implemented on WLS2)  
* Manifests from [Mepylome](https://pypi.org/project/mepylome) package.  
* IDAT pair named 'ref450k\_Grn.idat' and 'ref450k\_Red.idat' in the idat\_directory  
* data annotation file (described below)  
* reference data annotation file (described below)

## **Manifests, reference IDAT pair, and data annotations download**

* Download [data annotation file](https://docs.google.com/spreadsheets/d/e/2PACX-1vRhQ7Cr3aBo8W9Ne8DAehMvFRxYd395ENIW9giK2ATQ3QSrM8jA2E7xXbnW7CWKMdh0IhN0YqWn37Wr/pub?gid=0&single=true&output=csv) and [reference data annotation file](https://docs.google.com/spreadsheets/d/e/2PACX-1vRhQ7Cr3aBo8W9Ne8DAehMvFRxYd395ENIW9giK2ATQ3QSrM8jA2E7xXbnW7CWKMdh0IhN0YqWn37Wr/pub?gid=522048357&single=true&output=csv) data annotation sheets into the "diagnoses" directory specified in your .env file.  
* Download [manifests](https://epidip.usb.ch/cnquant/manifests.zip) and unzip it into the "manifests" directory specified in your .env file.  
* Download [reference IDAT pair](https://epidip.usb.ch/cnquant/ref450K.zip) and unzip it into the IDAT directory specified in your .env file.

## **Execution with CQmanager**

Instructions on how to use CQmanager for running CnQuant applications are in [CQmanager repository](https://github.com/neuropathbasel-pub/CQmanager).

Continue here [CnQuant Readme Draft 20250523 - part 2](https://docs.google.com/document/d/1dVb6fJx08Zx8QOxweAaVviR7a6TEPECIYAqiarVKPuM/edit?tab=t.0)