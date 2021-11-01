
# Case Studies

This repository provides R markdown templates to replicate the case
studies in the working paper *Generalized Nonlinear Models in Practice*
(Firth, Turner, and Kosmidis 2021). For most of the case studies, the
original data cannot be redistributed. Where possible, we provide
details of how to obtain the original data in this README, otherwise we
provide simulated data to produce results similar to those obtained with
the original data.

## Modelling Mortality Trends with the Lee-Carter Model

This is an application of the Lee-Carter model (Lee and Carter 1992) to
mortality rates of males in Canada between 1921 and 2003, obtained from
the [Human Mortality Database](http://www.mortality.org). Individuals
must register to gain access to the database - registration is free of
charge.

### Obtaining the Data

After registering as a user of the Human Mortality Database, go to the
homepage and select `Canada` from the table of countries. This opens a
page with a table of the available data sets. Under `Period data` select
`Deaths ... 1x1` and save the result as a `.txt` file, then select
`Exposure-to-risk ... 1x1` and save the result. Note the files will
contain data from 1921 up to the latest available date; the case study
in Firth, Turner, and Kosmidis (2021) uses data up to 2003.

### Reproducing the Analysis of Section 3

The analysis can be reproduced by rendering the `LeeCarter.Rmd` template
with the file paths to the downloaded data sets as parameters. For
example, if the deaths and exposures are saved as `Deaths_1x1.txt` and
`Exposures_1x1.txt` in the same directory as the R markdown template, it
may be rendered with the following code snippet:

``` r
rmarkdown::render("LeeCarter.Rmd", params = list(
  deaths = "Deaths_1x1.txt",
  exposures = "Exposures_1x1.txt"
), envir = new.env())
```

The `LeeCarter_original.html` file shows the results of rendering the
template with the original data.

Since the data can not be distributed here, simulated deaths and
exposures based on the Age-Period-Cohort generalized nonlinear model are
provided in `LeeCarter_artefacts/simulated_data`. The analysis template
can be rendered with these data as follows:

``` r
sim_dir <- file.path("LeeCarter_artefacts", "simulated_data")
rmarkdown::render("LeeCarter.Rmd", params = list(
  deaths = file.path(sim_dir, "Deaths_1x1.txt"),
  exposures = file.path(sim_dir, "Exposures_1x1.txt")
), output_file = "LeeCarter_simulated", envir = new.env())
```

### Reproducing the Figures of Section 3

The figures can be reproduced from saved results (stored in
`LeeCarter_artefacts/saved_results`) by not passing any data files to
the template:

``` r
rmarkdown::render("LeeCarter.Rmd",
       output_file = "LeeCarter_reproduced", envir = new.env())
```

``` r
checksum <- tools::md5sum(c("LeeCarter_original.html", 
                            "LeeCarter_reproduced.html"))
all.equal(checksum[["LeeCarter_original.html"]], 
          checksum[["LeeCarter_reproduced.html"]])
```

    [1] TRUE

## Comparing Social Mobility across Countries with the UNIDIFF Model

This is an application of the uniform difference or UNIDIFF model Xie
(1992) to data from a study of intergenerational social mobility in the
Netherlands, analysed previously in Ganzeboom and Luijkx (2004). The
data were obtained on request from the authors.

### Reproducing the Analysis of Section 4

The data are collated from 35 surveys conducted between 1974 and 1999.
The data for all men were provided as a data frame with 14 variables,
including

| Variable number | Value                                    |
|-----------------|------------------------------------------|
| 2               | Year                                     |
| 4               | 5-year period                            |
| 10              | Father’s social class (8 categories)     |
| 12              | Respondent’s social class (8 categories) |
| 14              | Weight frequencies                       |

The analysis can be reproduced by rendering the `UNIDIFF.Rmd` template
with the file path to this data file, for example:

``` r
rmarkdown::render("UNIDIFF.Rmd", params = list(
  data = "social_class.txt"
))
```

The `UNIDIFF_original.html` file shows the results of rendering the
template with the original data.

Since the data can not be distributed here, simulated data are provided
in `UNIDIFF_artefacts/simulated_data`, based on the UNIDIFF model with a
linear dependence of the log-multipliers on period. The analysis
template can be rendered with these data as follows:

``` r
sim_dir <- file.path("UNIDIFF_artefacts", "simulated_data")
rmarkdown::render("UNIDIFF.Rmd", params = list(
  data = file.path(sim_dir, "social_class.txt")
), output_file = "UNIDIFF_simulated", envir = new.env())
```

### Reproducing the Figures of Section 4

The figures can be reproduced from saved results (stored in
`UNIDIFF_artefacts/saved_results`) by not passing any data files to the
template:

``` r
rmarkdown::render("UNIDIFF.Rmd",
       output_file = "UNIDIFF_reproduced", envir = new.env())
```

``` r
checksum <- tools::md5sum(c("UNIDIFF_original.html", 
                            "UNIDIFF_reproduced.html"))
all.equal(checksum[["UNIDIFF_original.html"]], 
          checksum[["UNIDIFF_reproduced.html"]])
```

    [1] TRUE

## References

<div id="refs" class="references csl-bib-body hanging-indent">

<div id="ref-Erik92" class="csl-entry">

Erikson, R, and J H Goldthorpe. 1992. *<span class="nocase">The constant
flux</span>*. Oxford: Clarendon Press.

</div>

<div id="ref-GNMpaper" class="csl-entry">

Firth, David, Heather L Turner, and Ioannis Kosmidis. 2021. “Generalized
Nonlinear Models in Practice.”

</div>

<div id="ref-Ganz04" class="csl-entry">

Ganzeboom, Harry B. G., and Ruud Luijkx. 2004. “<span
class="nocase">Recent trends in intergenerational occupational class
reproduction in the Netherlands 1970-99</span>.” In *Social Mobility in
Europe*, edited by Richard Breen, 345–81. Oxford University Press.

</div>

<div id="ref-LeeCart92" class="csl-entry">

Lee, R D, and L Carter. 1992. “<span class="nocase">Modelling and
forecasting the time series of US mortality</span>.” *Journal of the
American Statistical Association* 87: 659–71.

</div>

<div id="ref-Xie92" class="csl-entry">

Xie, Y. 1992. “<span class="nocase">The log-multiplicative layer effect
model for comparing mobility tables</span>.” *American Sociological
Review* 57: 380–95.

</div>

</div>
