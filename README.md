# Source code for FRK v2

This repository contains the source code for reproducing the results in "Modelling Big, Heterogeneous, Non-Gaussian Spatial and Spatio-Temporal Data using FRK", which is in press with the *Journal of Statistical Software*. See [here](https://arxiv.org/abs/2110.02507) for a pre-print of the paper. 

## Creating a reproducible environment

There are three ways to create an environment that can reproduce the results of the paper, and these are listed below in the order of reliability:

1. using a docker container (cross platform),
1. using a conda environment (linux only),
1. using the current GitHub code + local install of current package versions. 

The first two options use package versions frozen at the time of submission, and are intended to ensure reproducibility for the foreseeable future. The third option is intended to provide users with a clear example of how our illustrative examples may be replicated using the latest versions of the associated packages. 

### Using a docker container

First, download and install `docker` or `Docker Desktop`. Then,

1. Download the archive containing all R packages, reproducible source code, and the Dockerfile from: https://hpc.niasra.uow.edu.au/FRKv2_docker.tar.gz
2. Uncompress the downloaded archive. This will yield a folder with the structure:  
```bash
├── FRKv2_docker
│   ├── Dockerfile
│   ├── FRKv2
```
3. Start a terminal and change directory to `FRKv2_docker`
4. Download the pre-prepared docker image from dockerhub: `sudo docker pull ycaodocker/frkv2`
5. Build a local docker image from the downloaded image: `sudo DOCKER_BUILDKIT=1 docker build -t 31b8d383613a:latest .`
6. Start a docker container with an interactive bash terminal (change `absolute_path` in the following command accordingly): `sudo docker run --rm -it -v "absolute_path/FRKv2_docker/FRKv2:/project" 31b8d383613a:latest bash`

Note that the above commands are for linux/mac, and small changes might be needed for Windows. For instance, it may be necessary to remove `sudo`, and replace `DOCKER_BUILDKIT=1` with `set DOCKER_BUILDKIT=1` in a separate line before building the docker file.

You are now ready to run the replication script described in the Results section. We suggest simply running `run_all.sh` (see below for details).


### Using a conda environment

The second method uses a conda environment to replicate the exact conditions at the time of submission (e.g., the version of R, package versions, etc.). This is a robust method to reproduce the results of the paper, but it is only available for Linux systems. First, download the packages and reproducible source code from https://hpc.niasra.uow.edu.au/FRKv2_conda.tar.gz, then enter the following commands:
```bash
tar -xzvf FRKv2_conda.tar.gz
cd FRKv2_conda
conda create -p .condaenv -c conda-forge gcc r-base=3.6.3 nlopt jpeg gmp gdal udunits2 proj
conda activate ./.condaenv
Rscript renv::rebuild('jpeg')
```
If the last command doesn't run, start `R` and run the command `renv::rebuild('jpeg')`.
You are now ready to run the replication script described in the Results section. We suggest simply running `run_all.sh` (see below for details).


### Current GitHub code + local install of current package versions

First, download this repo and navigate to its top-level directory within the command line.

Some data sets are too large (a few hundred Mb in total) to be stored on GitHub: These data sets are associated with Section 4.3 (Sydney spatial change-of-support) and Section 4.4 (Chicago crime). To download them and place them into the correct folders, run `bash DATA.sh`. If the data are not downloading as expected, please:

- Download the  [Chicago crime data set](https://zenodo.org/records/10278749) into the `data` folder; and
- download the [folders containing the Sydney SA1/SA2/SA3 region shapefiles](https://zenodo.org/records/10278720), unzip them, and move them into `data/Sydney_shapefiles`.

The folder `data` should contain the following files (as well as other files automatically downloaded from GitHub):

```bash
├── data
│   ├── chicago_crime_df.Rda
│   ├── Sydney_shapefiles
│   │   ├── SA1
│   │   ├── SA2
│   │   ├── SA3
```

Note that we have included checks at the beginning of the replication script to ensure that the user is immediately notified if these files are not present.


## Results

The replication script is `run_all.sh`, invoked using `bash run_all.sh`. Alternatively, Windows users may use `run_all.bat`. **NB**: the reproducible scripts ask the user if package dependencies should be installed - only do this if using the current GitHub code + local install option above. 

The replication script populates the `results` folder with the figures and tables given in the manuscript: These can then be viewed by opening `results_all.html` in any web browser. To quickly establish that the code is working, very-low-dimensional representations of the models can be used: The user is prompted for their choice when running the replication script. Note that some of the results will look very different when using these low-dimensional representations (particularly the results for Section 4.1).

Please note that the replication material for Section 3.3, Table 4, reproduces the results for row FRK v2 only; the results for other rows are taken from Table 3 in Heaton et al. (2019).

## Code structure

As described above, to reproduce the results of the manuscript, users need only run a single script, `run_all.{sh/bat}`, which runs the `R` files stored in the `scripts` folder: Each file corresponds to a single section of the manuscript, and is clearly labelled. There are a few small exceptions:

1. The fitting and prediction functions for each package in Section 4.1 (MODIS comparison) are stored in separate files, in the folder `scripts/MODIS_modelling_fns`. This was done so that the authors of these packages could quickly look to see how their package was used.
2. The file `Utility_fns.R` contains some basic helper functions used throughout the repo.
3. `setup.R` deals with the installation of dependencies listed in `dependencies.txt`, and checks that the data has been downloaded correctly.

We deviate from the standard of a single standalone replication script in part because of practical necessity; calling the `R` scripts individually from terminal means that each section is run independently of the other sections. This independence is important in order to keep the hardware requirements to a minimum; if the scripts are run in a single `R` session, garbage can accrue, leading to potential memory issues.   

## Hardware requirements

You will need at least 32GB of RAM (or RAM + swap) to run the very-low-dimensional representations of the models, and you will need at least 64GB of RAM (or RAM + swap) to run the full models.

## Run times

It takes a total of ~30 minutes to run the quick versions of the models. The following are estimates of the expected run time needed to reproduce the full results of the manuscript:

- Section 3.1 (Poisson simulation study):           ~5 minutes
- Section 3.2 (negative-binomial simulation study): ~5 minutes
- Section 3.3 (Heaton comparison study):            ~30 minutes
- Section 4.1 (MODIS comparison study):             ~15 hours
- Section 4.2 (Americium):                          ~5 minutes
- Section 4.3 (Sydney spatial change-of-support):   ~5 minutes
- Section 4.4 (Chicago crime):                      ~30 minutes

We note that the only problematic section is Section 4.1, the MODIS comparison study. This long run-time was unavoidable due to the nature of the study and the necessity to provide each package with every opportunity to perform well. In addition to the option of using low-dimensional representations as described above, one may easily reproduce the results from a subset of packages by editing the variable `PACKAGES` in `scripts/MODIS.R`. We thank reviewers for their patience and understanding.

Note that the replication script is clearly presented and commented; hence, one may easily "comment out" sections to produce a subset of the results. (Comments in `.sh` files are made with `##`, and comments in `.bat` files are made using `::`.)


## Real-world example

For users wishing to "try out" the package on a real-world example, we suggest the Sydney spatial change-of-support example presented in Section 4.3.
