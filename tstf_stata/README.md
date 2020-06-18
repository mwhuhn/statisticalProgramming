# Running TSTF on Stata with R

TSTF is a Stata module used to compute intervention time-series models. However, it relies on calling R, another statistical programming language. This guides walks you through installing and running TSTF on Stata and problems you might encounter. I am assuming you have Stata already installed.

## 1. Install R

First, you must install R and all the packages necessary for running the commands Stata calls in R. Choose a mirror [here](https://cran.r-project.org/mirrors.html) and install the appropriate version of R for your computer.

You will need to install a few packages (if not already installed): devtools, stats, foreign, aod, car, and TSA. Open R and run the following commands:

```R
install.packages("devtools")
install.packages("stats")
install.packages("foreign")
install.packages("aod")
install.packages("car")
```

The TSA package is no longer maintained and is archived on CRAN (Comprehensive R Archive Network) [here](https://cran.r-project.org/src/contrib/Archive/TSA/). We installed devtools so we can install TSA from the archive. Copy the URL for the version you want to use and paste it into the `install_url()` command:

```R
library(devtools)
install_url("https://cran.r-project.org/src/contrib/Archive/TSA/TSA_1.2.tar.gz")
```

## 2. Test the code in Stata

Once R is working properly, you can test tstf in Stata. If you haven't installed tstf yet, run:

```Stata
ssc install tstf
```

There are built-in data sets you can use to test tstf, including the Quitline data. The next bit of code will test to see if the tstf command is working properly. It opens the Quitline data, sets the data as a time series, and computes the model. While the command automatically selects a default location for your installed R program, it is a good idea to specify exactly where R is installed with `pathr()`. Below is the path to my R executable but yours will likely be in a different location even if you also run Windows.

```Stata
use http://www.stats4life.se/data/quitline, clear
tsset time, monthly
tstf lograte if inrange(time, 625, 691), smooth int(676) arima(1,1,1) sarima(1,0,0,12) pathr("C:/Program Files/R/R-3.6.1/bin/R.exe")
```

If this works, fantastic! You can use your own tstf data and get moving. Otherwise...

## 3. Version error

If you are running a version of Stata that is before Stata 14, the saveold command on line 210 of `tstf.ado` (on Windows, located at `C:/ado/plus/t`) will not work and you will give a version error. The original `tstf.ado` file can be edited so it works with pre-14 Stata. Just get rid of the version argument:

Before:

```Stata
quietly saveold `"data_tstf"' , replace nolabel version(11)
```

After:

```Stata
quietly saveold `"data_tstf"' , replace nolabel
```

Save the `tstf.ado` file and run the test code again. For extra safety, create a backup version of the `tstf.ado` file (`tstf-bkup.ado`) before you make any edits.

A copy of this edited file is `tstf-version.ado` in this folder. Make sure to restart Stata if you modify the package before trying to run the commands in part 2.

## 4. Parsing the generic error

The error message if something goes wrong is particularly unhelpful. If you look in the .ado file, any problem that doesn't return the coefficients file shows "the R script did not run, see what's wrong in the file `tstf_to_r.R`" as the error.

The first step is to see if Stata and R created the data files they use. If you look in your working directory and see data_tstf.dta but not `_tstf_b.dta`, `_tstf_stats.dta`, or `_tstf_V.dta` there was an error in R before it saved the data. Open the `tstf_to_r.R` file in your R interpreter and run it line-by-line. It might be that the required packages didn't install properly or you need to update R.

## 5. Errors after R

If R generates the new Stata datasets (`_tstf_b.dta`, `_tstf_stats.dta`, and `_tstf_V.dta`) but hits an error after loading them, you can run the `tstf.ado` file line-by-line until you find the error.

One problem I encountered was how R handles converting factors. If you get a string error when you try to decode the `names` variable in  `_tstf_b.dta`, R might have encoded `names` as a string rather than an int (as is assumed by the code). Replace line 275:

Before:

```Stata
qui decode names, gen(snames)
```

After:

```Stata
qui rename names snames
```

Save the `tstf.ado` file and run the test code again. For extra safety, create a backup version of the `tstf.ado` file (`tstf-bkup.ado`) before you make any edits.

A copy of this edited file is `tstf-string.ado` in this folder. Make sure to restart Stata if you modify the package before trying to run the commands in part 2.

## 6. Other

If you encounter errors using TSTF in Stata, let me know and I can update this guide!
