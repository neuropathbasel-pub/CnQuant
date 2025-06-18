# Table of contents

- [WSL2 image preparation](#wsl2-image-preparation)
- [CN analysis with CQcalc](#cn-analysis-with-cqcalc)
- [CN Summary Plots with CQall_plotter](#cn-summary-plots-with-cqall_plotter)
- [Per-case results inspection with CQcase](#per-case-results-inspection-with-cqcase)
- [CN Summary Plots inspection with CQall](#cn-summary-plots-inspection-with-cqall)

# WSL2 image preparation

>[!NOTE]
>All following commands are to be executed in PowerShell.</br>
>WSL2 has to be installed on your system.</br>
>Here is a [tutorial](https://www.windowscentral.com/how-install-wsl2-windows-10).

Steps:
1. Download the [WSL2 image](https://epidip.usb.ch/cnquant/cnquant.tar)
2. Import the WSL2 image:
Open PowerShell and run:

``` powershell
wsl --import cnquant windows_path_where_you_wish_to_keep_WSL_images windows_path_to_where_you_have_the_compressed_image\cnquant.tar
```

Replace
```
C:\path\to\store\wsl\images
```
with where you want to store the WSL2 image and  
```
C:\path\to\cnquant.tar
```
with the path to the downloaded image.

>[!TIP]
>You may change settings for CnQuant apps in WSL2 by modifying /home/cnquant/.env file.

3. Copy Sentrix ID pairs:
- Place your Sentrix ID pairs and reference IDAT files in:.
```
\\wsl.localhost\cnquant\home\cnquant\data\idat
```

4. Update the reference annotations in:
```
\\wsl.localhost\cnquant\home\cnquant\data\diagnoses\reference_data_annotation.csv
```

Add Sentrix IDs in the Sentrix_id column and references in the MC column.

# CN analysis with CQcalc

1. Analyze IDAT pairs in WSL2 image:</br>
with default settings:</br>
``` powershell
wsl -d cnquant --user cnquant -e bash -c "/home/cnquant/cqcalc/run_cqcalc.sh --sentrix_ids Sentix_ID1,Sentrix_ID2"
```
or with custom bin size, minimum probes per bin, or preprocessing method
``` powershell
wsl -d cnquant --user cnquant -e bash -c "/home/cnquant/cqcalc/run_cqcalc.sh --preprocessing_method illumina --bin_size 50000 --min_probes_per_bin 20 --sentrix_ids Sentix_ID1,Sentrix_ID2"
```

Following minimal and maximal bin size and minimum probes per bin settings:</br>
bin_size: minimum 1000, maximum 100000</br>
min_probes_per_bin: minimum 10, maximum 50

The CQcalc code allows to exceed those values but the execution might crash.

# CN Summary Plots with CQall_plotter

1. Update the data annotations in:
```
\\wsl.localhost\cnquant\home\cnquant\data\diagnoses\data_annotation.csv
```

Add Sentrix IDs in the Sentrix_id column and methylation class in the MC column.

2. Start CQall_plotter in WSL2 image with default settings:
``` powershell
wsl -d cnquant --user cnquant -e bash -c "/home/cnquant/cqall_plotter/run_cqall_plotter.sh"
```
or
Start CQall_plotter in WSL2 image with modified settings:
``` powershell
wsl -d cnquant --user cnquant -e bash -c "/home/cnquant/cqall_plotter/run_cqall_plotter.sh --preprocessing_method illumina --min_sentrix_ids_per_plot 5 --methylation_classes AML,GBM_RTKII"
```
where:</br>
--methylation_classes takes comma-separated list of methylation classes that are available in data_annotation.csv file in diagnoses directory.</br>
--preprocessing_method can be illumina or swan. Requires output from CQcalc.</br>
--min_sentrix_ids_per_plot defines minimum number of samples per methylation class to generate a plot.


# Per-case results inspection with CQcase

>[!NOTE]
>The port to access CQcase and CQall in the browser depends on the port set up in the .env file

1. Start CnQuant WSL2 image and CQcase:
``` powershell
wsl -d cnquant --user cnquant -e bash -c /home/cnquant/cqcase/run_cqcase.sh
```
2. Access CQcase in your browser
```
http:localhost:8062/cqcase
```
# CN Summary Plots inspection with CQall

>[!NOTE]
>The port to access CQcase and CQall in the browser depends on the port set up in the .env file

1. Start CnQuant WSL2 image and CQall:
``` powershell
wsl -d cnquant --user cnquant -e bash -c /home/cnquant/cqall/run_cqall.sh
```
2. Access CQcase in your browser
```
http:localhost:8060/cqall
```