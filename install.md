# Getting Started

The programs we will use in our tutorial today are are:

+  **_PathoScope_** (including _PathoLIB_,  _PathoMAP_ and _PathoID_)
+  **_BatchQC_**, and 
+  **_PathoStat_**

**_PathoScope_** is written in python and is used through the Terminal. On the other hand, **_BatchQC_** and **_PathoStat_** are interactive web applications written in R using the [Shiny](https://shiny.rstudio.com/) framework. There are quite a few sysadmin tasks needed to have everything working smoothly, so for today's tutorial, I've put together a Docker image so you can 


## Recommended Solution:

 [<img src="https://www.docker.com/sites/default/files/legal/small_h.png" width=200>](https://www.docker.com/what-docker)

#### Get Docker...

Docker is a software containerization platform. The basic idea is to wrap up a piece of software along with everything it needs to run. When you share this container with someone else (who is running Docker) they can load your container and the software will run the same.

If you don't have Docker yet, follow the instructions [here](https://www.docker.com/what-docker).

#### Now that we have Docker...

If you have a reasonable internet connection (~4GB download), you can pull the [image](https://hub.docker.com/r/mlbendall/pathosuite/) from Docker Hub:

```
docker pull mlbendall/pathosuite
```

Otherwise, you can load the image from a tarfile. (The image is included in today's tutorial, and is called `pathosuite.tar`.) Simply instruct docker to load the image from a file:

```
docker load -i pathosuite.tar
```

Check whether the image is installed by listing 

```
docker images
```

## Other Solutions

These other solutions do not provide the complete package, but may be useful if you already have some of the dependencies or you like your machine set up a certain way.


[<img src="https://bioconda.github.io/_static/logo/bioconda_monochrome_small.png" width=200>](https://bioconda.github.io/)

##### _Provides PathoScope, including PathoLib, PathoMap, and PathoID. Also installs bowtie2 and samtools dependencies._

_NOTE: The bioconda channel does not support windows builds._

Create a new `conda` environment containing python 2.7 and the pathoscope package:

```bash
conda create --name pathoscope python=2.7 pathoscope
```

Activate environment and test installation:

```bash
source activate pathoscope
pathoscope -h
```

[<img src="http://bioconductor.org/images/logo_bioconductor.gif" width=200>](http://bioconductor.org)

##### _Provides PathoStat and BatchQC, as well as any CRAN/Bioconductor depenedencies._

PathoStat was released with [Bioconductor 3.4](https://www.bioconductor.org/news/bioc_3_4_release/#new-software-packages). You need to have R 3.3 (>= 3.3.1 recommended) and may need to upgrade to the latest version.

```r
source("http://bioconductor.org/biocLite.R")
biocLite("BatchQC")
biocLite("PathoStat")
```

If you get an error, you may need to upgrade Bioconductor:

```r
biocLite("BiocUpgrade")
```


[<img src="https://assets-cdn.github.com/images/modules/logos_page/GitHub-Logo.png" width=200>](https://github.com)

We have detailed installation information for each package on their respective GitHub repositories:

[PathoScope](https://github.com/PathoScope/PathoScope)

[BatchQC](https://github.com/mani2012/BatchQC)

[PathoStat](https://github.com/mani2012/PathoStat)


<img src="https://c.s-microsoft.com/en-us/CMSImages/icon-windows-logo.svg?version=81bce4ab-327a-34bf-cf86-517ed6437fe9" width=200>

Finally, a few tips for Windows users if all else fails (courtsey of @ecastron):

[Installation on Windows](install_win.md)
