#  RcppDeepState tests
This repository contains the solution for the hard tests of the project "RcppDeepState" of GSOC 2022.
In this document we are using `RcppDeepState` library to analyze the [ldsr](https://cran.rstudio.com/web/packages/ldsr/index.html) package and compare with the [results provided](https://akhikolla.github.io./packages-folders/ldsr.html).

## Installation process
To install the package we can follow the classical devtools approach:
```R
install.packages("devtools")
require("devtools")
devtools::install_github("akhikolla/RcppDeepState")
```

Or in alternative we can download the source and install the package from a working R environment:
```R
install.packages("/path/to/RcppDeepState", repos = NULL, type="source")
```

## Installation issue 
After some checks I found that there was a problem in the source code of the `RcppDeepState` library. In particular in the method `deepstate_create_makefile` of the file `R/fun_makefile_create.R` at line *22*
```R
path <-paste("R_HOME=",dirname(R.home('include')))
```
should be changed to
```R
path <-paste("R_HOME=",R.home())
```
This change comes from the fact that `R.home('include')` returns the R installation path concatenated with the `include` directory. This variable `R_HOME` is then concatenated with another `include` at line *38*:
```bash
-I${R_HOME}/include
```
This will result in a double concatenation of the `include` keyword. 

#### My case example
In my case the result of `R.home('include')` is `/usr/include/R/` which doesn't contain any include directory: in fact it's the include directory itself.
The correct `R_HOME` directory location is correctly returned by using `R.home()` which returns in my case `/usr/lib64/R`. The latter concatenate with the `include` of line *38* is `/usr/lib64/R/include`, which is finally the correct path. 

## Tests execution
Now that the package is installed we can execute the `deepstate_harness_compile_run` to create the TestHarnesses for all the functions in the `ldsr` library. After downloading the source of [ldsr](https://cran.rstudio.com/web/packages/ldsr/index.html) from CRAN we can execute the compilation process and then analyze the results
```R
require("RcppDeepState")

deepstate_harness_compile_run("/path/to/ldsr_0.0.2")
result = deepstate_harness_analyze_pkg("/path/to/ldsr_0.0.2")
```

The list of compiled functions correspond to the one at the following [link](https://akhikolla.github.io./packages-folders/ldsr.html)
```
[1] "corr"  "KGE"   "nRMSE" "NSE"   "RE"
````
