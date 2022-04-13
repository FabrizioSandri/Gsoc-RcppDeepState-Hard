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

#### Solution
The solution to the problem is to add in the Makefile two variables
- one referring to the R home directory retrieved with the command `R.home()`
- one referring to the include path retrieved with the command `R.home("include")`

In fact by doing in this way we ensure that if the `include` directory isn't located in the R home directory the compilation process will still work. By doing in this way we avoided the use of concatenation.

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

#### Error comparison
The execution of the analysis on my computer reported only three errors out of four reported at this [link](https://akhikolla.github.io./packages-folders/ldsr.html). The three errors are reported in the following table, that is the content of the variable `result$logtable`

|err.kind|message|file.line|address.msg|address.trace|
|:---|:---|:---|:----|:---|
|UninitCondition |Conditional jump or move depends on uninitialised value(s) |utils.cpp : 37 |No Address found |No Address Trace found |
|InvalidRead |Invalid read of size 8 |utils.cpp : 56 |Address 0xb74e050 is 0 bytes after a block of size 624 alloc'd |utils.cpp : 55 |
|InvalidRead |Invalid read of size 8 |utils.cpp : 94 |Address 0x9c44d40 is 0 bytes after a block of size 320 alloc'd |NA|

As you can see the three errors correspond to the three provided [here](https://akhikolla.github.io./packages-folders/ldsr.html) : they occur in the same file, at the same line, and with the same error message. 

#### Strange behavior
After the first execution I only found 2 errors out of 4, so later on I decided to recompile everything with the same procedure described above. The result this time is that I was able to find 3 errors out of 4. It seems that the number of errors reported depends on some non-deterministic algorithms.

The most plausible motivation relies on the fact that fuzzing is a non-deterministic process where an application is sent an invalid set of inputs to check if there are some kind of errors. 




