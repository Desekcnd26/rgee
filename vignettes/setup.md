`rgee` has two types of dependencies: <span
style="color:#b52b09">**strict dependencies**</span> that must be
satisfied before the `rgee` installation and the <span style="color:#857e04"><b>credentials dependencies</b></span> that 
unlock `rgee` import & export functions.

If the first group of dependencies is not fulfilled **rgee just will not work**. The dependencies that comprised this group are:

-   <span style="color:#b52b09"><b> Google account with Earth Engine
    activated </b></span>
-   <span style="color:#b52b09"><b> Python >= v3.5 </b></span>
-   <span style="color:#b52b09"><b> EarthEngine Python API (Python package) </b></span>

The activation of **Earth Engine accounts** depends on each user, check
the oficial website of [Google Earth Engine](https://earthengine.google.com/) for more details. If you do not count with a Python environment or a version of the EarthEngine Python API, we strongly recommend you run  `ee_install(py_env = "rgee")`. This function will realize the following six tasks:
  
  1. If you do not count with a Python environment, it will display an
  interactive  menu to install [Miniconda](https://docs.conda.io/en/latest/miniconda.html)
  (a free minimal installer for conda).
  2. Remove the previous Python environment defined in py_env if it exist.
  3. Create a new Python environment.
  4. Set the environmental variable EARTHENGINE_PYTHON. It is used to define 
  RETICULATE_PYTHON when the library is loaded. See this article for further
  details.
  5. Install rgee Python dependencies. Using `reticulate::py_install`.
  6. Interactive menu to confirm if restart the R session to see changes.

However, the use of `ee_install()` is not mandatory. You can count on with your
own custom installation. This would be also allowed.

``` r
library(rgee)
ee_install() # It is just neccessary once!
```

By the other hand, the <span style="color:#857e04"><b>credentials dependencies</b></span>
are only needed to move data from Google Drive and Google Cloud Storage to your local environment.
These dependencies are not mandatory. However, they will help you to create a seamless connection between
R and Earth Engine.  The dependencies that comprised this group are shown below:

-   <span style="color:#857e04">**Google Cloud Storage
    credential**</span>
-   <span style="color:#857e04">**Google Drive credential**</span>

## Import and Export Spatial Data using rgee

The batch import/export involves difficulties for most GEE users. In
`rgee`, we are aware of it and we created several functions to help
users to download and upload spatial data. If you are trying to **export data** 
from Earth Engine, rgee offers the following options:

|         	|                   	|      FROM      	|       TO      	|       RETURN       	|
|---------	|-------------------	|:--------------:	|:-------------:	|:------------------:	|
| Image   	| ee_image_to_drive 	|    EE server   	|     Drive     	|   Unstarted task   	|
|         	| ee_image_to_gcs   	|    EE server   	| Cloud Storage 	|   Unstarted task   	|
|         	| ee_image_to_asset 	|    EE server   	|    EE asset   	|   Unstarted task   	|
|         	|   **ee_as_raster**  |    EE server   	|     Local     	| RasterStack object 	|
|         	|   **ee_as_stars**   |    EE server   	|     Local     	| Proxy-stars object 	|
| Table   	| ee_table_to_drive 	|    EE server   	|     Drive     	|   Unstarted task   	|
|         	| ee_table_to_gcs   	|    EE server   	| Cloud Storage 	|   Unstarted task   	|
|         	| ee_table_to_asset 	|    EE server   	|    EE asset   	|   Unstarted task   	|
|         	| **ee_as_sf**        |    EE server   	|     Local     	|      sf object     	|
| Generic 	| ee_drive_to_local 	|      Drive     	|     Local     	|   object filename  	|
|         	| ee_gcs_to_local   	|  Cloud Storage 	|     Local     	|     GCS filename  	|

: Table 1. Download functions provided by the rgee package.


In `rgee`, all the functions from server to local side have the option to fetch data using an intermediate container (Google Drive or Google Cloud Storage) or through a REST call ("\$getInfo"). Although the latter option performs a quick download, there is a request limit of 262144 pixels for ee\$Image and 5000 elements for ee\$FeatureCollection which makes it unsuitable for large objects. Other download functions, from server-side to others (see Table 1), are implemented to enable more customized download workflows. For example, using `ee_image_to_drive` and `ee_drive_to_local` users could create scripts which save results in the `.TFRecord` rather than the `.GeoTIFF` format. `rgee` to deal with **Google Drive** and **Google Cloud Storage** use the R package [googledrive](https://googledrive.tidyverse.org/) and [googleCloudStorageR](http://code.markedmondson.me/googleCloudStorageR/) respectively, so you will need to install it before.

``` r
# please try as follow
install.packages('googledrive')
install.packages('googleCloudStorageR')
```

**Google Drive** is more friendly to novice Earth Engine users because the
authentication process could be done without leaving R. However, if you
are trying to move large images or vectors, is preferable use **Google Cloud Storage** instead. For using Google Cloud Storage, you will need to have your own Google Project with a credit card added to the service, charges will apply. See the
[GCS\_AUTH\_FILE](https://github.com/csaybar/GCS_AUTH_FILE.json) tutorial to create your own service account key. If you want to understand why this is necessary, please have a look at [Mark Edmondson](http://code.markedmondson.me/googleCloudStorageR/articles/googleCloudStorageR.html)
tutorial.

Batch **upload** is a harder process, in `rgee` we try to make it
simple. If you want to upload files in a batch way, firstly you must
**get authorization to read & write into a Google Cloud Storage (GCS)
bucket**. `rgee` implement the next functions:


|         	|                 	|      FROM     	|       TO      	|            RETURN           	|
|---------	|-----------------	|:-------------:	|:-------------:	|:---------------------------:	|
| Image   	| gcs_to_ee_image 	| Cloud Storage 	|    EE asset   	|          EE Asset ID       	  |
|         	| raster_as_ee    	|     Local     	|    EE asset   	|          EE Asset ID       	  |
|         	| stars_as_ee     	|     Local     	|    EE asset   	|          EE Asset ID       	  |
| Table   	| gcs_to_ee_table 	| Cloud Storage 	|    EE asset   	|          EE Asset ID       	  |
|         	| sf_as_ee        	|     Local     	|    EE asset   	|          EE Asset ID       	  |
| Generic 	| local_to_gcs    	|     Local     	| Cloud Storage 	|         GCS filename        	|

: Table 2. Upload functions provided by the rgee package.

The upload process follows the same logic (Table 2). rgee includes `raster_as_ee` and `stars_as_ee` for uploading images and `sf_as_ee` for uploading vector data.

## Authentication

As we have seen previously, `rgee` deal with three different Google API’s:

-   Google Earth Engine
-   Google Drive
-   Google Cloud Storage

To authenticate either Google Drive or Google Cloud Storage, you just
need to run as follow:

``` r
library(rgee)
#ee_reattach() # reattach ee as a reserve word

# Initialize just Earth Engine
ee_Initialize() 
ee_Initialize(email = 'csaybar@gmail.com') # Use the argument email is not mandatory

# Initialize Earth Engine and GD
ee_Initialize(email = 'csaybar@gmail.com', drive = TRUE)

# Initialize Earth Engine and GCS
ee_Initialize(email = 'csaybar@gmail.com', gcs = TRUE)

# Initialize Earth Engine, GD and GCS
ee_Initialize(email = 'csaybar@gmail.com', drive = TRUE, gcs = TRUE)
```

If the Google account is verified and the permission is granted, you
will be directed to an authentication token. Copy this token and paste
it in the emerging GUI. Unlike Earth Engine and Google Drive, Google Cloud Storage need to set up its credential manually ([link1](http://code.markedmondson.me/googleCloudStorageR/articles/googleCloudStorageR.html) and [link2](https://github.com/csaybar/GCS_AUTH_FILE.json)). In all cases, the users credentials always will be stored in: 

``` r
ee_get_earthengine_path()
```
Remember you only have to authorize once, for next sessions it will not be necessary.

## Checking

The `ee_check()` function will help you for checking the sanity of your 
`rgee` installation and dependencies. 

-   `ee_check_python()` - Python version
-   `ee_check_credentials()` - Google Drive and GCS credentials
-   `ee_check_python_packages()` - Python packages

``` r
library(rgee)
ee_check()

ee_check_python()
ee_check_credentials()
ee_check_python_packages()
```
