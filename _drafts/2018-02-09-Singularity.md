---
title: HPC Computing Freedom with Singularity
author: Alexey Shiklomanov
date: 2018-02-09
layout: post
comments: true
---

# Singularity: What and Why?

Computing environments these days vary a lot.
As an extreme example, I run Arch Linux on my local laptop, which means I do most of my developmet on software that is bleeding edge...but my institutions high performance computing (HPC) environment runs CentOS 6, which runs software that can be a decade or more out of date.
Sometimes (suprisingly often, in fact), things work well enough even despite this discrepancy.
Often, though, such large discrepancies in software versions can cause things that work on my own machine to fail catastrophically and/or cryptically on the HPC and vice-versa.

Containerization was designed to solve this exact problem by providing a mechanism for distributing software and all of its dependencies neatly packaged together.
Perhaps the most popular piece of containerization software is Docker.
However, for various reasons, Docker is not well suited to HPC environments.

Enter Singularity. 
Singularity is container software similar in spirit (though very different in design and implementation) from Docker, but specifically designed to be less invasive and therefore more HPC-friendly.

# Case study: Custom R package

## Building the base R image

Start with a very simple recipe file called `base_recipe` (or whatever you like) containing just the following:

```
Bootstrap: docker
from: r-base:latest
```

Close the file and run the following command to build a container based on just this image.
Note that `sudo` is required here, so you'll have to do this in an environment over which you have administrative control.
(`sudo` is required whenever you are modifying the container, but not for actually using the container once it's been built.)

```
sudo singularity build base.simg base_recipe
```

This will take a while while the source files for the docker container are fetched and the container is being built.

When the process is finished, you can open up an interactive shell inside the container with the following:

```
singularity shell base.simg
```

Let's check the R version in here to make sure the container works as expected:

```
Singularity base.simg:~> R --version
```

This first container is not writeable.
By default, Singularity containers are read-only files.
This is by design -- they are meant to be treated as effectively self-contained binary executables, and you wouldn't want a binary executable to modify itself!

However, for debugging the development process, it is helpful to have a writable container.
To create a writable clone of this container, run the following command:

```
sudo singularity build --writable devel.simg base.simg
```

This will create a copy of the `base.simg` container called `devel.simg`, but which is writable.

The reason to create a writable container here is to allow you to try to setup a container interactively, and debug issues as they come up.
It's completely possible to create containers purely interactively, but it's strongly recommended (for provenance, clarity, and reproducibility) to create read-only containers entirely from a recipe file.

In a separate window, open the `base_recipe` file for editing.
Then, switch back to the terminal and open a shell inside the newly built `devel.simg` container:

```
sudo singularity shell devel.simg
# Singularity devel.simg:~>
```

Inside the container, let's try installing the useful `devtools` R package:

```
Singularity devel.simg:~> Rscript -e "install.packages('devtools')"
```

This command should fail because you are missing the OpenSSL dependency.
If we were building purely from a recipe file, this would abort the build and we would have to start over from scratch (fortunately, not quite from zero -- the docker base files are cached locally, but we would still have to unpack them and perform the other install steps).
However, interactively, we have an opportunity to debug this issue without starting over.

(Note that you have to restart from zero only if the build fails.
Images created from successful builds can be "built into" -- this means that all of the commands in the `%post` section are re-run again, but on top of the existing container, so, for instance, commands like `apt-get install ...` will still be run but, because the targets are already installed, they will take dramatically less time
This also means that you need to be careful with `%post` commands like `git clone` that will error if the target already exists).

This particular bug is easy to address.
Since the `r-base` container is Ubuntu-based, we can install the dependency with the following `apt` commands:

```
Singularity devel.simg:~> apt-get update
Singularity devel.simg:~> apt-get install -y libssl-dev
```

Now that you know these commands are necessary to install the R packages you need, you should add them to your recipe file.
Installation steps like this should come in the `%post` section, so your `base_recipe` file should now look like this:

```
Bootstrap: docker
from: r-base:latest

%post
    apt-get update
    apt-get install -y libssl-dev
```

Switch back to the terminal and try installing the `devtools` package again.
This time, there should be a different missing system dependency.
Install it, add the installation step to the `recipe_file`, and repeat until you have the components you need.

Skipping ahead a bit, to install the `devtools`, `roxygen2`, and `testthat` packages, I have a `base_recipe` that looks like this:

```
Bootstrap: docker
from: r-base:latest

%post
    apt-get update
    apt-get install -y \
        libssl-dev \
        libcurl4-openssl-dev \
        libxml2-dev \
        git

    Rscript -e "install.packages('devtools')"
    Rscript -e "devtools::install_cran('roxygen2')"
    Rscript -e "devtools::install_cran('testthat')"
```

(Note the use of backslashes to escape line breaks, which allow me to make a more readable list of system dependencies. Also note the `-y` flag to skip the `apt` `yes/no` install prompt.)

At this point, you can extend this container with more stuff if you'd like.
However, because Singularity makes it easy to build containers from other containers, I would recommend building subsequent, analysis-specific containers off of this image, to save time on all the complex installation steps (though, for reproducibility, you should keep a readily accessible copy of the `base_recipe` on hand).
Personally, I make widespread use of the `tidyverse` packages, so to save time, I have added that as an installation step to my `base_recipe`.

Once you are happy with the contents of your base container recipe, exit the Singularity shell (Control-D or shell command `exit`), delete the `base.simg` file, and try to rebuild it using the new recipe:

```
rm base.simg
sudo singularity build base.simg base_recipe
```

If this succeeds (which it should, if you've been diligent about documenting your steps), proceed to the next step:

## Creating the analysis-specific container

For this test case, my demo uses a plant trait analysis I did for a _New Phytologist_ paper.
However, the principles are pretty general and should work equally well for just about any project.

First, create a new recipe file called `mvtraits_recipe` with the following contents:

```
Bootstrap: localimage
from: base.simg
```

This header indicates we are building from a local image `base.simg`, located in the current directory.

Before adding any more to the recipe file, let's create a writable image we can experiment with and enter it:

```
sudo singularity build --writable mvtraits_devel.simg mvtraits_recipe
# ...build process...
sudo singularity shell mvtraits_devel.simg
```

The first step is to install my `mvtraits` R package from GitHub.
Because we already have `devtools` installed, we can just do the following:

```
S...> Rscript -e "devtools::install_github('ashiklom/mvtraits')"
```

(Note, for brevity, from now on I'll be abbreviating the Singularity prompt to `S...>`).

This will fail because `mvtraits` depends on the `RcppZiggurat` package, which depends on the `gsl` system library (i.e. the Ubuntu `libgsl-dev` package).
Install this library with `apt-get install libgsl-dev`, and add the corresponding entry to the `mvtraits_recipe` file, which should now look like this:

```
Bootstrap: localimage
from: base.simg

%post
    apt-get install -y \
        libgsl-dev

    Rscript -e "devtools::install_github('ashiklom/mvtraits')"
```

The `mvtraits` package should now install successfully.
If it does, let's now clone the more specific analysis repository, which is also an R package but also contains ancillary data and run scripts.

```
S...> git clone git://github.com/ashiklom/np_trait_analysis analysis
```

(Note the use of the `git` protocol in the URL -- this is somewhat insecure but also is the least likely to run into errors with missing system libraries, certificates, etc. Also, it's worth noting that because it's insecure, the `git` protocol prevents pushing, which we won't need anyway.)

Now, install the R package components of this repository (remember to add these commands to the `mvtraits_recipe` file as well):

```
S...> cd analysis
S...> Rscript -e "devtools::install('.')"
# ...build process...
S...> cd ..
```

This will install a few more R packages as dependencies (which in turn may have system dependencies -- if so, follow steps like those described above to install them).

The final step is to specify what the container will actually do when we run it as an executable.
To do this, add the following block to the `mvtraits_recipe` file:

```
%runscript
    Rscript --vanilla /analysis/scripts/run_model.R $@
```

This indicates that, by default, running the container will run the `analysis/scripts/run_model.R` script.
Note the use of an absolute path -- everything in the Singularity container is done relative to its root (`/`) directory.
The `$@` ensures that all command line arguments passed to 

The final version of the `mvtraits_recipe` file will look like this:

```
Bootstrap: localimage
from: base.simg

%post
    apt-get install -y \
        libgsl-dev

    Rscript -e "devtools::install_github('ashiklom/mvtraits')"

    if [ ! -d analysis ]; then
        git clone git://github.com/ashiklom/np_trait_analysis analysis
    fi

    cd analysis
    Rscript -e "devtools::install('.')"
    cd ..

%runscript
    Runscript /analysis/scripts/run_model.R $@
```

## TODO:
- Successful builds can be extended (build "into" containers) -- don't have to start from scratch if build is successful.
