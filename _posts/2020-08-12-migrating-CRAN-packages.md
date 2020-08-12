---
layout: post
title: "Migrating CRAN Packages after R Upgrades"
date: 2020-08-12
author: Paul Egeler, M.S.
categories: jekyll update
---

## Background

This blog post is a supplement to the West Michigan R User Group
[March 2020 Meeting]({% post_url 2020-03-02-meeting-announcement-2020-03 %}).

## Introduction

You may have noticed that sometimes after upgrading to a new version of 
[R](https://cran.r-project.org/), you need to install all of your packages
again. This happens because the new install of R can't find the location of the
old packages. Even if it could, it would still be a good idea to rebuild all of
your packages because packages are built for specific versions of R. So
sometimes a package built for one version will need to be rebuilt for another.
I'm leaving out a lot of details here, but that is the gist of it.

When I upgrade my version of R, many scripts will start breaking since R can't
find required packages on my system. Then I have to go track down which packages
are missing in my new environment and install them manually. This can take a
long time, especially on systems where you must compile from source. I would
like to have all the same packages avialable to me that I had in my previous
version so that everything just works out of the box. What is the best way to do
that? I'm not sure, but I have whipped up a solution that has worked for me,
although it has its flaws.

## My Current-State Workflow

Since the majority of the packages that I use are from CRAN, I have written an R
script that goes through a library location to identify and install CRAN packages
while telling me which packages I will need to install from elsewhere (_e.g._, 
[Bioconductor](https://www.bioconductor.org/),
[Omegahat](http://www.omegahat.net/), 
[github](https://github.com/SpectrumHealthResearch), 
internal and personal packages).

I still install non-CRAN pacakges manually, but it reduces the number of
packages that I have to process considerably. Installing non-CRAN packages could
be automated to a degree, but since packages are installed from so many
different places, I have not seen the benefit.

Here are the simple steps that I use:

### Finding the Library Path

Prior to installing a new version of R, I find out where my existing version of
R is searching to find its packages. These are the locations that I will be
coming back to in order to get a list of packages to build under my new version
of R. To do that, I use the `.libPaths()` function.

In particular, the first element returned is my personal library (this
may also be defined by the `R_LIBS_USER` environment variable). Unless you are a
site administrator, that is likely where the packages you want will be. I take
note of these locations, as I will need them in the next step.

### After Upgrade

Now that I have the location to the user library of my prior installation, I'm
ready to upgrade. Once I've have upgraded, I start an R session and `source()`
this file:

<script src="https://gist.github.com/pegeler/9c45b7766bd70cd44836752dae3b4077.js"></script>

The function `install_packages_from_library()` takes a vector of library paths
to check and the dots (`...`) are passed to `install.packages()` so that I can
pass custom parameters such as a CRAN mirror or specify building from source.

An example usage would be:

```r
install_packages_from_library('/home/user1/R/x86_64-pc-linux-gnu-library/3.5')
```

I get one or two prompts and then package installation commences.

## Alternative Approaches

The internet is full of 
[articles](https://shiny.rstudio.com/articles/upgrade-R.html),
[blog posts](https://www.r-bloggers.com/automated-re-install-of-packages-for-r-3-0/),
[Stack Overflow answers](https://stackoverflow.com/questions/1401904/painless-way-to-install-a-new-version-of-r),
and [forum posts](https://www.reddit.com/r/rstats/comments/67a9dh/how_do_you_migrate_your_packages_after_an_update/)
on the subject.

A very popular answer seems to be to copy all packages from the old library 
directory to the new one and then run 
`update.packages(ask = FALSE, checkBuilt = TRUE)`
(note the `checkBuilt` parameter, which will cause the function to
upgrade packages built under a different version). This seems like a very
rational and convenient solution that would work for most cases. However, a
quick glance at the function documentation and source code indicates to me that
it does not solve the problem of packages from non-standard sources. As a
result, the `update.packages()` approach risks moving packages into your new R
library that were built for the wrong version of R. Hence my reason for creating
the script above.

I still think there must to be a better way. How can I automate installation of
_all_ packages in a library, not just CRAN packages? Also, is there a better way
to standardize my environment across workstations? For example, what do I do
when I move to a new machine which does not have access to those old package
libraries?

### Metapackage, or One Package to Install Them All

That got me thinking about the power of the packages themselves. Specifically,
metapackages, or packages that install/invoke other packages. A popular
metapackage that you probably already know is
[`tidyverse`](https://cran.r-project.org/package=tidyverse). So why not a
metapackage for personal use? Or even metapackages for specific use-cases?

With the 
[`Remotes` field](https://cran.r-project.org/web/packages/devtools/vignettes/dependencies.html)
in a package _DESCRIPTION_ file, even packages from non-standard sources can be
installed automatically with [`devtools`](https://cran.r-project.org/package=devtools).

So with that, I have created [my first metapackage](https://github.com/pegeler/metapackage).

## Conclusion

There does not seem to be a clear consensus on exactly the right way to handle
package migration between versions of R. My previous workflow has served me well,
but I am looking forward to using my new metapackage in the future. We will soon
see how fastidous I am about updating packages that I begin using more or removing
those that fall by the wayside.

## Open Forum

What is your strategy?
