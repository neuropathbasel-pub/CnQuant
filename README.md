# Table of contents

- [Disclamer](#disclamer)
- [Introduction to the app](#introduction-to-the-app)
- [Requirements](#requirements)
    - [Directory structure and data annotation files](#directory-structure-and-data-annotation-files)
    - [Software requirements](#software-requirements)
    - [Manifests, reference IDAT pair, and data annotations download](#manifests-reference-IDAT-pair-and-data-annotations-download)
- [Execution with CQmanager](#execution-with-cqmanager)
- [Running CnQuant apps in WSL2](#running-CnQuant-apps-in-wsl2)

# Disclaimer

Implementation of this software in a diagnostic setting occurs in the sole responsibility of the treating physician.
Usage of this software occurs at the risk of the user. The authors may not be held liable for any damage (including hardware) this software might cause.
Use is explicitly restricted to academic and non-for-profit organizations.

# Introduction to the app

CnQuant, is a lightweight stack of 5 applications designed to streamline the analysis of Illumina Infinium Methylation array data through a web browser
CnQuant consists of the following programs (stored in separate repositories) for CNV calculation and visualization:

- [CQmanager](https://github.com/neuropathbasel-pub/CQmanager)
- [CQcalc](https://github.com/neuropathbasel-pub/CQcalc)
- [CQall plotter](https://github.com/neuropathbasel-pub/CQall_plotter)
- [CQcase](https://github.com/neuropathbasel-pub/CQcase)
- [CQall](https://github.com/neuropathbasel-pub/CQall)

CnQuant seamlessly integrates raw data from multiple kit versions (450k, EPICv1, and EPICv2),
merging it into an Illumina Infinium Methylation array 450k format for consistent downstream processing.
Leveraging this harmonized data, [CQcalc](https://github.com/neuropathbasel-pub/CQcalc) computes copy number variants (CNVs)
and [CQcase](https://github.com/neuropathbasel-pub/CQcase) visualizes these results for every processed IDAT pair,
offering clinicians clear, actionable insights.
Beyond case-based CNV analysis, [CQall plotter](https://github.com/neuropathbasel-pub/CQall_plotter)
generates tumor entity-specific summary plots that highlight recurrent CNV patterns.
The summary plots are visualized with [CQall](https://github.com/neuropathbasel-pub/CQall).
In sum, CnQuant simplifies complex workflows, delivering robust results in an intuitive package.
CnQuant is optimized for low-latency data visualization and inspection.
The CNV data is pre-analyzed on a local server and stored in parquet files for rapid viewing and examination through the web-based frontends [CQcase](https://github.com/neuropathbasel-pub/CQcase) and [CQall](https://github.com/neuropathbasel-pub/CQall).

# Requirements

If you do not have Docker installed on your system, proceed to Docker installation documentation for your platform.</br>
After the installation, ensure to carry out the post-installation steps to run the container as a non-root user.
If you are not familiar with docker, you might be interested in [Docker Curriculum](https://docker-curriculum.com/).

Another option to run the CnQuant apps is WSL2 subsystem for Windows, described in [Running CnQuant apps in WSL2](#running-cnquant-apps-in-wsl2).

The last option (not recommended) is to install the applications with pip
but it will require R-4.3.0 installation with CnQuant R dependencies,
which is described separately in each of the applications repository.

## Directory structure and data annotation files

For convenience, the same .env file is used for each app.
The .env file needs to be located in each apps' main directory.
An "example.env" file is provided in each application's directory.
Modifying .env files for WSL2 is typically not necessary.

## Environment variables

Before running any of the 5 applications (not needed for WSL2),
you will need to create the following directories and define them in your .env file.
Download [example.env](https://github.com/neuropathbasel-pub/CnQuant/blob/main/example.env) into your environment directory where you wish to install one of the CnQuant apps,
rename it to .env and adjust the paths inside the .env file.

>[!Note]
>On some operating systems, visual file explorers (Thunar, Gnome, Explorer) do not display files starting with a "." in their filename by default.</br>
>Refer to your system's manual on how to display .env if need be.

**Local directories**
- log_directory: log files in plain text format will be saved here
- idat_directory: this directory contains raw IDAT file pairs (*_Red.idat and *_Grn-idat). Subfolders will not recursively be checked for IDAT pairs.
- reference_directory: directory, where preprocessed reference methylation data will be stored in a binary format.
- diagnoses_directory: directory which has to contain two comma-separated value (CSV) files: data_annotation.csv with two headers, Sentrix_id and MC, which will contains annotated Sentrix IDs and their user-defined methylation class. The file reference_data_annotation.csv has the same structure with Sentrix IDs annotated as "reference"
- cn_results_directory: directory where CNV results will be stored in compressed parquet format.
- summary_plots_base_directory: summary plots will be stored here.
- temp_directory: this directory will be used to store temporary files, which, during normal operation, should be automatically removed if no longer required.
- manifests_directory: this directory will contains Illumina manifests for array conversion and analysis

**Remote server directories**

The following remote directories are for running CQcase and CQall on a remote server.
CQmanager runs CQcalc on the same server where you run CQmanager.
CQmanager does not sync data.
The remote server could be a resource-limited system in a DMZ to provide public or institutional web interface access,
while the other CnQuant applications would probably run on a system with larger resources.

- remote_server_log_directory: log files from apps running on a remote server via CQmanager in plain text form will be saved here.
- remote_server_idat_directory: this directory on a remote server must contain raw IDAT file pairs (Red and Green). Subfolders will not be checked for existing IDATs. For apps running on a remote server via CQmanager.
- remote_server_reference_directory: directory on a remote server, where preprocessed reference methylation data will be stored in binary format. For apps running on a remote server via CQmanager.
- remote_server_diagnoses_directory: directory on a remote server which has to contain two csv files: data_annotation.csv with two headers Sentrix_id and MC, which will contain annotated Sentrix IDs and their annotated tumor types. The second file, reference_data_annotation.csv, has the same structure with Sentrix IDs annotated as "reference".
- remote_server_cn_results_directory: directory on remote directory where CNV results will be stored in compressed parquet format. For CQcase running on a remote server via CQmanager.
- remote_server_summary_plots_base_directory: summary plots will be stored here on the remote server. For CQall running on a remote server via CQmanager.

**CQcalc-specific settings**
- process_inconvertible_sentrix_ids: set this to True to allow reprocessing Sentrix IDs that were previously determined to be corrupt. Default is False. Note: Depending on the amount of IDAT files present on a system, reprocessing many files can require significant amounts of time and resources.
- check_if_idats_have_equal_size: set this to True to reject IDAT pairs with unequal file size. Default is True. Note: Valid IDAT files are typically in the same size range.
- minimum_idat_size: set here minimum idat size in MB per file. Default is 1. Note: For more stringent filtering (many IDAT file types are larger than 1 MB) increase according to expected file size.

**CQall_plotter-specific settings**
- minimal_number_of_sentrix_ids_for_summary_plot: controls minimum number of analyzed samples per summary plot. Default is 5.

**Server name displayed in CQcase and CQall**
- server_name: A string or sentence wrapped in double quotes you put here will be displayed in bottom right corner in CQcase and CQall

**CQcase- and CQall-specific settings**
- cqcase_host_app_port: port where you will be able to access CQcase. Default is 8052
- cqall_host_app_port: port where you will be able to access CQall. Default is 8050
- email_notification_port: Port for SMTP server for email notifications. Default is 587
- workers: Number of workers for [Gunicorn](https://gunicorn.org/) for running CQcase and CQall. Default is 10, suitable for multiprocessor systems. If you are running the apps on a desktop PC for a single user only, set it to 1.
- timeout: Timeout for caching. Default is 300. Unit is seconds.
- REDIS_HOST=localhost: host for [Redis](https://redis.io/) caching. Default is localhost
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
- CQviewers_host: Your remote server host name or ip.
- CQviewers_user: Your remote server username. This user needs to be able to run Docker containers.
- CQall_container_name: Name for CQall container for Docker naming purposes. Default is cqall.
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
- DATA_ANNOTATION_SHEET: URL for google sheet csv output for annotations
- REFERENCE_DATA_ANNOTATION_SHEET: URL for google sheet csv output for reference annotations
- URL Example: https://docs.google.com/spreadsheets/d/e/2PACX-1vRhQ7Cr3aBo8W9Ne8DAehMvFRxYd395ENIW9giK2ATQ3QSrM8jA2E7xXbnW7CWKMdh0IhN0YqWn37Wr/pub?gid=0&single=true&output=csv


## Software requirements
- Ubuntu 20.04 or 22.04 (other distributions have not been tested)
- Python 3.10, 3.11, or 3.12
- Docker (recommended) or WLS2 (CQmanager is not implemented on WLS2)
- Manifests from [Mepylome](https://pypi.org/project/mepylome) package.
- IDAT pair named 'ref450k_Grn.idat' and 'ref450k_Red.idat' in the idat_directory
- data annotation
- reference data annotation

## Manifests, reference IDAT pair, and data annotations download

- Download [data](https://docs.google.com/spreadsheets/d/e/2PACX-1vRhQ7Cr3aBo8W9Ne8DAehMvFRxYd395ENIW9giK2ATQ3QSrM8jA2E7xXbnW7CWKMdh0IhN0YqWn37Wr/pub?gid=0&single=true&output=csv) and [reference](https://docs.google.com/spreadsheets/d/e/2PACX-1vRhQ7Cr3aBo8W9Ne8DAehMvFRxYd395ENIW9giK2ATQ3QSrM8jA2E7xXbnW7CWKMdh0IhN0YqWn37Wr/pub?gid=522048357&single=true&output=csv) data annotation sheets into the "diagnoses" directory specified in your .env file.
- Download [manifests](https://epidip.usb.ch/cnquant/manifests.zip) and unzip it into the "manifests" directory specified in your .env file.
- Download [reference IDAT pair](https://epidip.usb.ch/cnquant/ref450K.zip) and unzip it into the IDAT directory specified in your .env file.

## Execution with CQmanager

For automated executions with CQmanager instructions in [CQmanager](https://github.com/neuropathbasel-pub/CQmanager) repository

# Running CnQuant apps in WSL2

## Prepare WSL2 image
Steps:
1. Download the [WSL2 image](https://epidip.usb.ch/cnquant/cnquant.tar)
2. Import the WSL2 image:
Open PowerShell and run:

``` powershell
wsl --import cnquant windows_path_where_you_wish_to_keep_WSL_images windows_path_to_where_you_have_the_compressed_image\cnquant.tar
```

Replace C:\path\to\store\wsl\images with where you want to store the WSL2 image and C:\path\to\cnquant.tar with the path to the downloaded image.

3. Copy Sentrix ID pairs:
- Place your Sentrix ID pairs and reference IDAT files in the same directory.
- Copy them to:

```
\\wsl.localhost\cnquant\home\cnquant\data\idat
```

- Update the reference annotations in:
```
\\wsl.localhost\cnquant\home\cnquant\data\diagnoses\reference_data_annotation.csv
```
Add Sentrix IDs in the Sentrix_id column and references in the MC column.

4. Start CnQuant applications in WSL2 image:
In PowerShell:

``` powershell
wsl -d cnquant --user cnquant --cd /home/cnquant/cqcalc -e "source /home/cnquant/cqcalc/.venv/bin/activate && cqcalc \
--preprocessing_method illumina \
--bin_size 50000 \
--min_probes_per_bin 20 \
--sentrix_ids 'your_sentrix_id1,your_sentrix_id2,your_sentrix_id3'"
```

``` powershell
wsl -d cnquant --user cnquant --cd /home/cnquant/cqall -e bash -c "source /home/cnquant/cqall/.venv/bin/activate"  && echo aaa && run_cqall
wsl -d cnquant --user cnquant --cd /home/cnquant/cqall -e bash -c "source /home/cnquant/cqall/.venv/bin/activate && exec bash"
wsl -d cnquant --user cnquant --cd /home/cnquant/cqall -e bash -c "source /home/cnquant/cqall/.venv/bin/activate && export POLARS_ALLOW_FORKING_THREAD=1 && run_cqall && exec bash"
```

``` powershell
wsl -d cnquant --user cnquant --cd /home/cnquant/cqcase -e source /home/cnquant/cqcalc/.venv/bin/activate
```

``` powershell
wsl -d cnquant --user cnquant --cd /home/cnquant/cqall_plotter -e source /home/cnquant/cqcalc/.venv/bin/activate
```

5. Activate the virtual environment:
``` bash
source .venv/bin/activate
```
6. Analyze your samples:
``` bash
cqcalc \
--preprocessing_method illumina \
--bin_size 50000 \
--min_probes_per_bin 20 \
--sentrix_ids 'your_sentrix_id1,your_sentrix_id2,your_sentrix_id3'
```
7. Inspect results:
Use [CQcase](https://github.com/neuropathbasel-pub/CQcase) in WSL2 to review your data.

>[!NOTE]
>Invalid IDAT pairs are logged in cnv_conversion_register.csv in the logs directory. To retry processing, add the --rerun_sentrix_ids flag to the cqcalc command.

# Detailed instructions for separate applications

- [CQmanager](https://github.com/neuropathbasel-pub/CQmanager)
- [CQcalc](https://github.com/neuropathbasel-pub/CQcalc)
- [CQall plotter](https://github.com/neuropathbasel-pub/CQall_plotter)
- [CQcase](https://github.com/neuropathbasel-pub/CQcase)
- [CQall](https://github.com/neuropathbasel-pub/CQall)