
# RStudio-FIELDimageR

FIELDimageR in a RStudio container with `VICE` dependencies, based on [Vice RStudio Docker container](https://hub.docker.com/r/cyversevice/rstudio-verse) for CyVerse VICE.

## Overview

The original documentation for [FIELDimageR](https://github.com/OpenDroneMap/FIELDimageR) provides information on its capabilities.
It is strongly recommended that you read the documentation provided by FIELDimageR before generating the GeoJSON as described in this document.

This documentation is for writing [GeoJSON](https://datatracker.ietf.org/doc/rfc7946/?include_text=1) plot files using the functionality provided by FIELDimageR.
If you have access to an existing VICE app you can skip ahead to the [vice section](#vice).

There is a non-CyVerse Docker image available named [agdrone/fieldimager](https://hub.docker.com/repository/docker/agdrone/fieldimager).
Refer to the [rocker/rstudio](https://hub.docker.com/r/rocker/rstudio) instructions for running this Docker image.
The instructions starting at the [Preparation Steps](#preparation) section can be used with this image.

# Quick Start

Use the following steps to quickly get started with the CyVerse app:
1. Log into [CyVerse](https://de.cyverse.org/de/) Discovery Environment
2. Click on the Apps tile to open the `Apps` window
3. If needed, find the FIELDimageR app
4. Click on the name of the app to open the FIELDimageR analysis launch window
5. Specify a folder containing the source image(s) in the `Sources` section of the analysis launch window
6. Click the `Launch Analysis` button to start the app
7. Click on the `Analysis` tile to open that window
8. Click the `Go to analysis` image link next to the running FIELDimageR instance (which opens a new window) and wait until the CyVerse login screen appears
9. Log into CyVerse with your credentials
10. If prompted, log into the RStudio instance using the username `rstudio` and password `rstudio1` credentials
11. Follow the instructions in the [preparation step](#preparation) to generate the plot GeoJSON file

# Docker Instructions

## Run this Docker locally or on a Virtual Machine

To run these containers, you must first pull them from DockerHub.

```
docker pull agdrone/cyverse_fieldimager:latest
```

```
docker run --rm -v /$HOME:/app --workdir /app -p 8787:80 -e REDIRECT_URL=http://localhost:8787 agdrone/cyverse_fieldimager:latest
```

#### Local Credentials

You may be prompted for a username and password.
If you are **not** prompted, you will be automatically logged into the RStudio instance.

The default username is `rstudio` and password is `rstudio1`.
To use a different password, add the flag `-e PASSWORD=<yourpassword>` in the `docker run` statement (replacing \<yourpassword\> with the actual password).

## Build your own Docker container and deploy on CyVerse VICE

A pre-built image is available on [DockerHub](https://hub.docker.com/repository/docker/agdrone/cyverse_fieldimager).
This pre-built image is intended to run on the CyVerse data science workbench, called [VICE](https://cyverse-visual-interactive-computing-environment.readthedocs-hosted.com/en/latest/index.html). 

##### Developer notes

To build your own container with a Dockerfile and additional dependencies, get the [Dockerfile](https://github.com/Chris-Schnaufer/rstudio-FIELDimageR) from the GitHub repository.

Next, [build](https://docs.docker.com/engine/reference/commandline/build/) the Docker image using a command line prompt.

A sample command line is shown next; you will need to replace parameters for your build environment.
```bash
docker build -t agdrone/cyverse_fieldimager:1.0 -f 1.0.0/Dockerfile ./
```
The following parameters are:
* `docker` this is the Docker command to run
* `build` instructions docker to build an image
* `-t` indicates that the tag (name) of the image follows
* `agdrone/cyverse_fieldimager:1.0` the tag of the built image; this name should be changed for your environment
* `-f` indicates that the path of the Dockerfile to use follows
* `1.0.0/Dockerfile` the relative path to the Dockerfile to use; the path to the Dockerfile may be different on your system
* `./` indicates that the current folder is to be used as needed

Once the Docker image is built, [docker push](https://docs.docker.com/engine/reference/commandline/push/) can be used to upload the image to where CyVerse VICE can access it.
The following command uploads the image built in the previous step to [DockerHub](https://hub.docker.com/).
You will need to replace the image tag in the following command with the one specified previously in the "docker build" command.
```bash
docker push agdrone/cyverse_fieldimager:1.0
```

Follow the instructions in the [VICE manual for integrating your own tools and apps](https://cyverse-visual-interactive-computing-environment.readthedocs-hosted.com/en/latest/developer_guide/building.html).

# Running the VICE app <a name="vice"/>

Additional information is available in the CyVerse [RStudio Quick Start](https://learning.cyverse.org/projects/vice/en/latest/user_guide/quick-rstudio.html) documentation. 

#### VICE Credentials

An RStudio username and password is not needed when running the latest Docker image on VICE.
If prompted, the RStudio username is `rstudio` and password is `rstudio1`.

# Generating Plot GeoJSON File
This document describes how to use [FIELDimageR](https://github.com/OpenDroneMap/FIELDimageR) to generate a plot boundary [GeoJSON](https://datatracker.ietf.org/doc/rfc7946/?include_text=1) file.

These instructions assume that you have access to a running Docker container that already has the requirements for FIELDimageR installed, along with additional packages for working with JSON.
Refer to the [installation](https://github.com/OpenDroneMap/FIELDimageR#Instal) instructions if needed.
The CyVerse FIELDimageR app comes with all the requirements pre-installed.

We will only be referencing the minimum steps needed for generating the GeoJSON file.
There is additional functionality available with FIELDimageR.

## Preparation Steps <a name="preparation" />

This section lists the steps needed to define the plot boundaries ([saving the plot geometries](#generate) follows).

<hr />

The steps used here are a non-consecutive subset of those available with FIELDimageR with additional commands added.

- [2. Select the targeted field from the original image](https://github.com/OpenDroneMap/FIELDimageR#2-selecting-the-targeted-field-from-the-original-image)
- [3. Rotating the image](https://github.com/OpenDroneMap/FIELDimageR#3-rotating-the-image)
- [5. Building the plot shape file](https://github.com/OpenDroneMap/FIELDimageR#5-building-the-plot-shape-file)

<hr />

We are using an example source image named [EX1_RGB.tif](https://drive.google.com/file/d/1S9MyX12De94swjtDuEXMZKhIIHbXkXKt/view).
You can substitute the name of any georeferenced image file you'd like.

**Specify libraries needed**

Load the needed libraries
```R
library(FIELDimageR)
library(raster)
library(geojsonio)
library(stringi)
```

There may be messages indicating dependent packages are loaded and that some functions are masked.
For the purposes of these instructions, those messages can be ignored.

**Load the image**

Load the source image with the following command.
Remember to substitute the name of your image for `EX1_RGB.tif`.
```R
EX1<-stack("EX1_RGB.tif")
```

**Crop to field of interest**

Run the following step to crop the image to the field.
This will make it easier to specify the plot boundaries.
```R
EX1.Crop <- fieldCrop(mosaic = EX1)
```

**Rotate the image**

The cropped image needs to be rotated so that the columns are vertically aligned (facing "up").
The `clockwise` parameter is shown as having a `F` (False) value indicating counter-clockwise rotation.
If you need clockwise rotation, or your attempts to vertically align the columns isn't working out, try setting the parameter to `T` (True).
If your row/column orientation is rotated, you can set the `h` parameter to `T` (True) to indicate the orientation.

For our purposes the `extentGIS` parameter must be set to `T` or `TRUE`.
```R
source(system.file("extdata", "fieldRotate.R", package = "FIELDimageR")) # Needed for GIS support
EX1.Rotated<-fieldRotate(mosaic = EX1.Crop, clockwise = F, h = F, extentGIS = T)
```

*Important: Be sure to note the displayed `Theta rotation` value which we need later.*

**Defining the plot boundaries**

Now that the image is prepared, the plot boundaries can be defined.

Replace the parameter assignment value `<Theta Rotation>` with the actual theta rotational value noted in the previous step. 

Be sure to specify the correct count of columns (`ncols`) and rows (`nrows`) for your field.

```R
EX1.Shape<-fieldShape(mosaic = EX1.Rotated, ncols = 14, nrows = 9, theta = <Theta Rotation>)
```

## Generate GeoJSON file <a name="generate" />

The following code takes the plot boundaries produced in the [Preparation](#preparation) section above and writes the GeoJSON to a file named 'plots.geojson'.

Ensure coordinate system of polygons is in the expected Lat-Long WGS84 (EPSG 4326).
```R
latlon_crs<-crs('+proj=longlat +datum=WGS84 +no_defs')
latlon_shape<-spTransform(EX1.Shape$fieldShapeGIS, latlon_crs)
```

Generate and write the JSON:
```R
json<-geojson_json(latlon_shape)
save_json<-gsub('fieldID', 'id', json)
geojson_write(save_json, geometry="polygon", file="plots.geojson", precision=9)
```

## Finishing up

After [generating](#generate) the `plots.geojson` file, it is now available on the file system.
You are able to view and export/download this file from within the running RStudio instance.

Within CyVerse you are able to `Complete and save outputs`, which will place the `plots.geojson` file in the associated analysis folder.
