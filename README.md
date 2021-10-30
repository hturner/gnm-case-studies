
# Case Studies

This repository provides R markdown templates to replicate the case
studies in the working paper *Generalized Nonlinear Models in Practice*
(Firth, Turner, and Kosmidis 2021). For most of the case studies, the
original data cannot be redistributed. Where possible, we provide
details of how to obtain the original data in this README, otherwise we
provide simulated data to produce results similar to those obtained with
the original data.

## Modelling Mortality Trends with the Lee-Carter Model

This is an application of the Lee-Carter model Lee and Carter (1992) to
mortality rates of males in Canada between 1921 and 2003, obtained from
the [Human Mortality Database](http://www.mortality.org). Individuals
must register to gain access to the database - registration is free of
charge.

### Obtaining and Pre-processing the Data

After registering as a user of the Human Mortality Database, go to the
homepage and select `Canada` from the table of countries. This opens a
page with a table of the available data sets. Under `Period data` select
`Deaths ... 1x1` and save the result as a `.txt` file, then select
`Exposure-to-risk ... 1x1` and save the result. Note the files will
contain data from 1921 up to the latest available date; the case study
in Firth, Turner, and Kosmidis (2021) uses data up to 2003.

### Reproducing the Analysis of Section 3.1

The analysis can be reproduced by rendering the `LeeCarterCaseStudy.Rmd`
template with the file paths to the downloaded data sets as parameters.
For example, if the deaths and exposures are saved as `Deaths_1x1.txt`
and `Exposures_1x1.txt` in the same directory as the R markdown
template, it may be rendered with the following code snippet:

``` r
rmarkdown::render("LeeCarter.Rmd", params = list(
  deaths = "Deaths_1x1.txt",
  exposures = "Exposures_1x1.txt"
))
```

The `LeeCarter_original.html` file shows the results of rendering the
template with the original data.

Since the data can not be distributed here, simulated deaths and
exposures based on the Age-Period-Cohort generalized nonlinear model are
provided in `LeeCarter_artefacts/simulated_data`. The analysis template
can be rendered with this data as follows:

``` r
sim_dir <- file.path("LeeCarter_artefacts", "simulated_data")
rmarkdown::render("LeeCarter.Rmd", params = list(
  deaths = file.path(sim_dir, "Deaths_1x1.txt"),
  exposures = file.path(sim_dir, "Exposures_1x1.txt")
), output_file = "LeeCarter_simulated.html")
```

### Reproducing the Figures of Section 3.1

The figures can be reproduced from saved results (stored in
`LeeCarter_artefacts/saved_results`) by not passing any data files to
the template:

``` r
rmarkdown::render("LeeCarter.Rmd",
       output_file = "LeeCarter_reproduced.html")
```

``` r
checksum <- tools::md5sum(c("LeeCarter_original.html", 
                            "LeeCarter_reproduced.html"))
all.equal(checksum[["LeeCarter_original.html"]], 
          checksum[["LeeCarter_reproduced.html"]])
```

    ## [1] TRUE

## References

<div id="refs" class="references csl-bib-body hanging-indent">

<div id="ref-GNMpaper" class="csl-entry">

Firth, David, Heather L Turner, and Ioannis Kosmidis. 2021. “Generalized
Nonlinear Models in Practice.”

</div>

<div id="ref-LeeCart92" class="csl-entry">

Lee, R D, and L Carter. 1992. “<span class="nocase">Modelling and
forecasting the time series of {US} mortality</span>.” *Journal of the
American Statistical Association* 87: 659–71.

</div>

</div>
