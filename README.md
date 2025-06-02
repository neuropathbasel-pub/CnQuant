# Table of contents

- [Disclaimer](#disclamer)
- [Introduction to the app](#introduction-to-the-app)
- [Requirements](#requirements)
    - [Directory structure and data annotation files](#directory-structure-and-data-annotation-files)
    - [Software requirements](#software-requirements)
    - [Manifests, reference IDAT pair, and data annotations download](#manifests-reference-IDAT-pair-and-data-annotations-download)
- [Execution with CQmanager](#execution-with-cqmanager)
- [Running CnQuant apps in WSL2](#running-CnQuant-apps-in-wsl2)
- [Alternative ways to install and run CnQuant apps](#alternative-ways-to-install-and-run-cnquant-apps)

# Disclaimer

Implementation of this software in a diagnostic setting occurs in the sole responsibility of the treating physician.
Usage of this software occurs at the risk of the user. The authors may not be held liable for any damage (including hardware) this software might cause.
Use is explicitly restricted to academic and not-for-profit organizations.

# Introduction

CnQuant, is a lightweight stack of 5 applications designed to streamline the analysis of Illumina Infinium Methylation array data
through a web browser CnQuant consists of the following programs (stored in separate repositories) for CNV calculation and visualization:

- [CQmanager](https://github.com/neuropathbasel-pub/CQmanager)
- [CQcalc](https://github.com/neuropathbasel-pub/CQcalc)
- [CQall plotter](https://github.com/neuropathbasel-pub/CQall_plotter)
- [CQcase](https://github.com/neuropathbasel-pub/CQcase)
- [CQall](https://github.com/neuropathbasel-pub/CQall)

CnQuant seamlessly unifies raw data from multiple kit versions (450k, EPICv1, and EPICv2) by merging it into an Illumina Infinium Methylation array 450k format for downstream processing.
Based on the harmonized data, [CQcalc](https://github.com/neuropathbasel-pub/CQcalc) computes copy number variants (CNVs)
and [CQcase](https://github.com/neuropathbasel-pub/CQcase) visualizes these results for every processed IDAT pair,
displaying potentially actionable targets in point-of-care settings.
Beyond case-specific CNV analysis, the [CQall plotter](https://github.com/neuropathbasel-pub/CQall_plotter) plotter computes tumor entity-specific summary plots from case collections,
which highlights recurrent CNV patterns.
Summary plots are visualized with [CQall](https://github.com/neuropathbasel-pub/CQall).
In sum, CnQuant simplifies complex workflows, delivering robust results in short time in an intuitive package.
CnQuant is optimized for low-latency data visualization and inspection.
Microarray data are pre-analyzed on a local server and stored in Parquet files for rapid viewing and examination through the web-based frontends [CQcase](https://github.com/neuropathbasel-pub/CQcase) and [CQall](https://github.com/neuropathbasel-pub/CQall).

# Requirements

A Docker installation is required.
Ensure to carry out the Docker post-installation steps to run the container as a non-root user.
If new to Docker, the [Docker Curriculum](https://docker-curriculum.com/) might be helpful.
In case of running CnQuant applications on a server without internet access, you will need to download all docker images elsewhere and transfer them to your Docker host:

``` bash
docker pull \
neuropathologiebasel/cqcalc:latest \
neuropathologiebasel/cqcase:latest \
neuropathologiebasel/cqall:latest \
neuropathologiebasel/cqall_plotter:latest

docker save -o cnquant_images.tar \
neuropathologiebasel/cqcalc:latest \
neuropathologiebasel/cqcase:latest \
neuropathologiebasel/cqall:latest \
neuropathologiebasel/cqall_plotter:latest

# Before this command, the cnquant_images.tar needs to be transferred
# to the host machine without internet connection

docker load -i cnquant_images.tar
```

A Docker alternative is  WSL2 (Windows subsystem for Linux 2), described in [Running CnQuant apps in WSL2](#running-cnquant-apps-in-wsl2).
Primarily intended for developers or embedded systems with limited resources or CPUs other than x86_64,
all CnQuant components can be installed from source with pip.
Note that CnQuant depends on both Python and R, requiring R-4.3.0 (for [CQcalc](https://github.com/neuropathbasel-pub/CQcalc)) in addition to an appropriate Python environment.
Installation instructions are described separately in each application component repository.


## Directory structure and data annotation files

For convenience, the same .env file is used for each app.
The .env file needs to be located in each apps' main directory or its parent directory.
The .env file in an application directory has precedence before the .env file in the parent's directory.
An "example.env" file is provided in each application's directory or in the parent's directory.
Adapting .env files for WSL2 is typically not necessary.

## Environment variables

Before running any of the 5 applications (not needed for WSL2),
you will need to create the following directories and define them in your .env file.
Download [example.env](https://github.com/neuropathbasel-pub/CnQuant/blob/main/example.env) into your environment directory or in it's parent directory where you wish to install one of the CnQuant apps,
rename it to .env and adjust the paths inside the .env file.

>[!Note]
>Some visual file explorers (Thunar, Gnome, Explorer) do not display files starting with a "." in their filename by default and might require settings adjustment.

**Local directories**
- log_directory: log files in plain text format will be saved here.
- idat_directory: contains raw IDAT file pairs (*_Red.idat and *_Grn-idat). Subdirectories will not recursively be checked for IDAT pairs.
- reference_directory: preprocessed reference methylation data will be stored here in a binary format.
- diagnoses_directory: has to contain two comma-separated value (CSV) files: data_annotation.csv with two headers, Sentrix_id and MC. data_annotation.csv contains annotated Sentrix IDs and their user-defined methylation classes. The file reference_data_annotation.csv has the same structure.
- cn_results_directory: CNV results will be stored here in Parquet format.
- summary_plots_base_directory: summary plots will be stored here.
- temp_directory: used to store temporary files, which, during normal operation, should be automatically removed if no longer required.
- manifests_directory: contains Illumina manifest files for array conversion and analysis

**Remote server directories**

The following remote directories are required for running CQcase and CQall on a remote server.
CQmanager launches CQcalc on the same server where you run CQmanager.
CQmanager does not sync data.
A remote server could be a resource-limited system in a DMZ to provide public or institutional web interface access (CQcase and CQall),
while the remaining CnQuant are background applications intended to run on a system with larger resources.

- remote_server_log_directory: log files from apps running on a remote server via CQmanager in plain text form will be saved here.
- remote_server_diagnoses_directory: directory on a remote server which has to contain two csv files: data_annotation.csv with two headers Sentrix_id and MC, which will contain annotated Sentrix IDs and their annotated tumor types. The second file, reference_data_annotation.csv, has the same structure with Sentrix IDs annotated as "reference".
- remote_server_cn_results_directory: directory on remote directory where CNV results will be stored in compressed parquet format. For CQcase running on a remote server via CQmanager.
- remote_server_summary_plots_base_directory: summary plots will be stored here on the remote server. For CQall running on a remote server via CQmanager.

**CQcalc-specific settings**

-process_inconvertible_sentrix_ids: set this to True to allow reprocessing Sentrix IDs that were previously determined to be corrupt. Default is False. Note: Depending on the amount of IDAT files present on a system, reprocessing many files can require significant amounts of time and resources.
-check_if_idats_have_equal_size: set this to True to reject IDAT pairs with unequal file size. Default is True. Note: Valid IDAT files are typically in the same size range.
-minimum_idat_size: set here minimum IDAT size in MB per file. Default is 1. Note: For more stringent filtering (many IDAT file types are larger than 1 MB) increase according to expected file size.

**CQall_plotter-specific settings**
- minimal_number_of_sentrix_ids_for_summary_plot: controls minimum number of analyzed samples per summary plot. Default is 5.

**Server name displayed in CQcase and CQall**
- server_name: A string or sentence wrapped in double quotes you put here will be displayed in the bottom right corner in CQcase and CQall.

**CQcase- and CQall-specific settings**
- cqcase_host_app_port: port where you will be able to access CQcase. Default is 8052
- cqall_host_app_port: port where you will be able to access CQall. Default is 8050
- email_notification_port: The port for SMTP server for email notifications. Default is 587
- workers: Number of workers for [Gunicorn](https://gunicorn.org/) for running CQcase and CQall. Default is 10, suitable for multiprocessor systems. If you are running the apps on a desktop PC for a single user only, set it to 1.
- timeout: Timeout for caching. Default is 300. Unit is seconds.
- REDIS_HOST: host for [Redis](https://redis.io/) caching. Default is localhost.
- REDIS_PORT: Port for Redis caching. Default is 6379.
- maximum_number_of_genes_to_plot: Maximum number of additional genes plotted in CQcase and CQall. Default is 600. Note that plotting too many genes will make the plot quite unreadable and will slow down the application.
- CACHING_DB_cqcase: Redis caching database number for CQcase. Default is.
- CACHING_DB_cqall: Redis caching database number for CQall. Default is 1.

**Docker images**
- cqcalc_image: CQcalc Docker image name. Default is neuropathologiebasel/cqcalc:latest
- cqcase_image: CQcase Docker image name. Default is neuropathologiebasel/cqcase:latest
- cqall_image: CQall Docker image name. Default is neuropathologiebasel/cqall:latest
- cqall_plotter_image: CQall plotter Docker image name. Default is neuropathologiebasel/cqall_plotter:latest

**CQmanager-specific settings. To control containerized CQviewers on remote server, setting up ssh keys is needed**
- CQmanager_gunicorn_host_address: Address where to run CQmanager. Default is 127.0.0.1.
- CQmanager_gunicorn_port: Port on which to run CQmanager. Default is 8002.
- cnv_register_file_name: File name for a register of processed Sentrix IDs. Default is cnv_conversion_register.csv.
- CQ_manager_batch_size: Number of Sentrix IDs per CQcalc container. Default is 100. Due to R memory leaks, it is recommended to tune this parameter moderately. Depends on the available RAM and CPU threads.
- CQ_manager_batch_timeout: Seconds after CQmanager starts, a container that did not reach set batch size. Default is 300.
- max_number_of_cqcalc_containers: Maximum number of CQcalc containers running in parallel. Default is 10. Depends on your available RAM and CPUs.
- run_CQviewers_on_remote_server: Set it to True if you wish to run CQcase and CQall on a remote server. Default is False. Remember that you will need to set up a ssh key and provide username and host. Docker needs to be installed on the host.
- CQviewers_host: Your remote server host name or IP.
- CQviewers_user: Your remote server username. This user needs to be able to run Docker containers.
- CQall_container_name: Name for CQall container for Docker container naming purposes. Default is cqall.
- CQcase_container_name: Name for CQcase container for Docker naming purposes. Default is cqcase.
- cnquant_redis_name: Name for Redis container for Docker naming purposes. Default is cnquant_redis.
- CQviewers_docker_network_name: Name for the network that will be created for CQcase and CQall. Default is cnquant_network.
- initiate_cqcase_and_cqall_on_startup: Set it to True if you wish to start CQcase and CQall with Redis containers on CQmanager startup. Default is False.
- remote_user_id: remote user ID for executing containers with. Required for file permissions.
- remote_group_id: remote group ID for executing containers with. Required for file permissions.

**Email notification settings**
- crash_email_sender: sender email address for notification emails in case of a crash. Works with Gmail.
- crash_email_receivers: Comma-separated list of email recipient email addresses for crash notifications.
- crash_email_sender_password: Application password for sender's email address. Wrap it in double quotes.

**Data annotation URLs**
- DATA_ANNOTATION_SHEET: URL for google sheet csv output for annotations.
- REFERENCE_DATA_ANNOTATION_SHEET: URL for google sheet csv output for reference annotations.
- URL Example: https://docs.google.com/spreadsheets/d/e/2PACX-1vRhQ7Cr3aBo8W9Ne8DAehMvFRxYd395ENIW9giK2ATQ3QSrM8jA2E7xXbnW7CWKMdh0IhN0YqWn37Wr/pub?gid=0&single=true&output=csv


## Software requirements
- Ubuntu 20.04 or 22.04 (other distributions have not been tested)
- Python 3.10, 3.11, or 3.12
- Docker (recommended) or WLS2 (CQmanager is not implemented on WLS2)
- Manifests from [Mepylome](https://pypi.org/project/mepylome) package.
- IDAT pair named 'ref450k_Grn.idat' and 'ref450k_Red.idat' in the idat_directory
- data annotation file (described below)
- reference data annotation file (described below)

## Manifests, reference IDAT pair, and data annotations download

- Download [data annotation file](https://docs.google.com/spreadsheets/d/e/2PACX-1vRhQ7Cr3aBo8W9Ne8DAehMvFRxYd395ENIW9giK2ATQ3QSrM8jA2E7xXbnW7CWKMdh0IhN0YqWn37Wr/pub?gid=0&single=true&output=csv) and [reference data annotation file](https://docs.google.com/spreadsheets/d/e/2PACX-1vRhQ7Cr3aBo8W9Ne8DAehMvFRxYd395ENIW9giK2ATQ3QSrM8jA2E7xXbnW7CWKMdh0IhN0YqWn37Wr/pub?gid=522048357&single=true&output=csv) data annotation sheets into the "diagnoses" directory specified in your .env file.
- Download [manifest](https://epidip.usb.ch/cnquant/cnquant_manifest.parquet) into the "manifests" directory specified in your .env file.
- Download [reference IDAT pair](https://epidip.usb.ch/cnquant/ref450K.zip) and unzip it into the IDAT directory specified in your .env file.

## Execution with CQmanager

>[!Note]
>If you are using Docker Desktop, you might need to make the host directories available to Docker containers, as set up in .env file.</br>
>You can find these options in Docker Desktop Settings -> Resources -> File sharing -> Virtual file shares.

Instructions on how to use CQmanager for running CnQuant applications are in [CQmanager repository](https://github.com/neuropathbasel-pub/CQmanager).

# Running CnQuant apps in WSL2

For instructions about CnQuant apps execution in WSL2, please continue [here](./README_WSL2.md)


# Alternative ways to install and run CnQuant apps

Alternative way of installation and running CnQuant apps are described in separately in the apps' repositories

- [CQmanager](https://github.com/neuropathbasel-pub/CQmanager)
- [CQcalc](https://github.com/neuropathbasel-pub/CQcalc)
- [CQall plotter](https://github.com/neuropathbasel-pub/CQall_plotter)
- [CQcase](https://github.com/neuropathbasel-pub/CQcase)
- [CQall](https://github.com/neuropathbasel-pub/CQall)