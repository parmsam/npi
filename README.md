
<!-- README.md is generated from README.Rmd. Please edit that file -->

# npi

> Access the U.S. National Provider Identifier Registry
API

[![lifecycle](https://img.shields.io/badge/lifecycle-maturing-blue.svg)](https://www.tidyverse.org/lifecycle/#maturing)
[![Travis build
status](https://travis-ci.org/frankfarach/npi.svg?branch=master)](https://travis-ci.org/frankfarach/npi)
[![AppVeyor Build
Status](https://ci.appveyor.com/api/projects/status/github/frankfarach/npi?branch=master&svg=true)](https://ci.appveyor.com/project/frankfarach/npi)
[![Coverage
status](https://codecov.io/gh/frankfarach/npi/branch/master/graph/badge.svg)](https://codecov.io/github/frankfarach/npi?branch=master)

Use R to access the U.S. National Provider Identifier (NPI) Registry API
(v2.1) by the Center for Medicare and Medicaid Services (CMS):
<https://npiregistry.cms.hhs.gov/>. Obtain rich administrative data
linked to a specific individual or organizational healthcare provider,
or perform advanced searches based on provider name, location, type of
service, credentials, and many other attributes. `npi` provides
convenience functions for data extraction so you can spend less time
wrangling data and more time putting data to work.

## Installation

Install `npi` directly from Github using the `devtools` package:

``` r
devtools::install_github("frankfarach/npi")
library(npi)
```

## Usage

`npi` exports four functions, all of which match the pattern "npi\_\*":

  - `npi_search()`: Search the NPI Registry and return the response as a
    [tibble](http://tibble.tidyverse.org/) with high-cardinality data
    organized into list columns.
  - `npi_summarize()`: A method for displaying a nice overview of
    results from `npi_search()`.
  - `npi_flatten()`: A method for flattening one or more list columns
    from a search result, joined by NPI number.
  - `npi_is_valid()`: Check the validity of one or more NPI numbers
    using the official [NPI enumeration
    standard](https://www.cms.gov/Regulations-and-Guidance/Administrative-Simplification/NationalProvIdentStand/Downloads/NPIcheckdigit.pdf).

### Search the registry

`npi_search()` exposes nearly all of the NPPES API’s [search
parameters](https://npiregistry.cms.hhs.gov/registry/help-api). Let’s
say we wanted to find up to 10 organizational providers with primary
locations in New York City:

``` r
nyc <- npi_search(city = "New York City")
```

``` r
nyc
#> # A tibble: 10 x 11
#>       npi enumeration_type basic other_names identifiers taxonomies
#>  *  <int> <chr>            <lis> <list>      <list>      <list>    
#>  1 1.40e9 Individual       <tib… <tibble [0… <tibble [0… <tibble […
#>  2 1.34e9 Individual       <tib… <tibble [0… <tibble [0… <tibble […
#>  3 1.15e9 Individual       <tib… <tibble [0… <tibble [1… <tibble […
#>  4 1.37e9 Individual       <tib… <tibble [0… <tibble [0… <tibble […
#>  5 1.97e9 Individual       <tib… <tibble [0… <tibble [0… <tibble […
#>  6 1.16e9 Individual       <tib… <tibble [0… <tibble [0… <tibble […
#>  7 1.35e9 Organization     <tib… <tibble [0… <tibble [1… <tibble […
#>  8 1.59e9 Organization     <tib… <tibble [0… <tibble [3… <tibble […
#>  9 1.75e9 Organization     <tib… <tibble [0… <tibble [0… <tibble […
#> 10 1.02e9 Organization     <tib… <tibble [1… <tibble [2… <tibble […
#> # … with 5 more variables: addresses <list>, practice_locations <list>,
#> #   endpoints <list>, created_date <dttm>, last_updated_date <dttm>
```

The full search results have four regular vector columns, `npi`,
`provider_type`, `created_date`, and `last_updated_date` and seven list
columns. Each list column is a collection of related data:

  - `basic`: Basic profile information about the provider
  - `other_names`: Other names used by the provider
  - `identifiers`: Other provider identifiers and credential information
  - `taxonomies`: Service classification and license information
  - `addresses`: Location and mailing address information
  - `practice_locations`: Provider’s practice locations
  - `endpoints`: Details about provider’s endpoints for health
    information exchange

If you’re comfortable [working with list
columns](https://r4ds.had.co.nz/many-models.html), this may be all you
need from the package. But let’s not stop just yet, because `npi`
provides convenience functions to summarize and extract the data you
need.

## Working with search results

Run `npi_summarize()` on your results to see a more human-readable
overview of what we’ve got:

``` r
npi_summarize(nyc)
#> # A tibble: 10 x 6
#>        npi name   enumeration_type primary_practice… phone primary_taxonomy
#>      <int> <chr>  <chr>            <chr>             <chr> <chr>           
#>  1  1.40e9 AMIR … Individual       10 E 102ND ST, N… 212-… Internal Medici…
#>  2  1.34e9 CLAUD… Individual       1540 E COLORADO … 818-… Social Worker C…
#>  3  1.15e9 PREMI… Individual       90 S BEDFORD RD,… 914-… Family Medicine 
#>  4  1.37e9 JEFFR… Individual       1501 N CAMPBELL … 520-… Internal Medici…
#>  5  1.97e9 MARK … Individual       1400 BELLINGER S… 715-… Hospitalist     
#>  6  1.16e9 STACE… Individual       10 E 102ND ST, N… 212-… Internal Medici…
#>  7  1.35e9 NICUL… Organization     10 EAST 38TH STR… 212-… Internal Medici…
#>  8  1.59e9 UNDER… Organization     160 E 34TH ST 4T… 212-… Durable Medical…
#>  9  1.75e9 NEURO… Organization     635 WEST 165 ST,… 212-… Ophthalmology   
#> 10  1.02e9 MEDIC… Organization     250 EAST HOUSTON… 212-… Podiatrist
```

Suppose we just want the basic and taxonomy information for each NPI in
the result in a flattened data frame:

``` r
npi_flatten(nyc, c("basic", "taxonomies"))
#> # A tibble: 19 x 27
#>       npi basic_name_pref… basic_first_name basic_last_name
#>     <int> <chr>            <chr>            <chr>          
#>  1 1.02e9 <NA>             <NA>             <NA>           
#>  2 1.15e9 <NA>             PREMILA          MATHEWS        
#>  3 1.15e9 <NA>             PREMILA          MATHEWS        
#>  4 1.16e9 <NA>             STACEY-ANN       BROWN          
#>  5 1.16e9 <NA>             STACEY-ANN       BROWN          
#>  6 1.16e9 <NA>             STACEY-ANN       BROWN          
#>  7 1.16e9 <NA>             STACEY-ANN       BROWN          
#>  8 1.34e9 MS.              CLAUDIA          NARVAEZ-MEZA   
#>  9 1.34e9 MS.              CLAUDIA          NARVAEZ-MEZA   
#> 10 1.35e9 <NA>             <NA>             <NA>           
#> 11 1.35e9 <NA>             <NA>             <NA>           
#> 12 1.37e9 DR.              JEFFREY          SHRENSEL       
#> 13 1.37e9 DR.              JEFFREY          SHRENSEL       
#> 14 1.37e9 DR.              JEFFREY          SHRENSEL       
#> 15 1.40e9 DR.              AMIR             STEINBERG      
#> 16 1.59e9 <NA>             <NA>             <NA>           
#> 17 1.75e9 <NA>             <NA>             <NA>           
#> 18 1.97e9 DR.              MARK HENRY       ALON           
#> 19 1.97e9 DR.              MARK HENRY       ALON           
#> # … with 23 more variables: basic_middle_name <chr>,
#> #   basic_credential <chr>, basic_sole_proprietor <chr>,
#> #   basic_gender <chr>, basic_enumeration_date <chr>,
#> #   basic_last_updated <chr>, basic_status <chr>, basic_name <chr>,
#> #   basic_organization_name <chr>, basic_organizational_subpart <chr>,
#> #   basic_authorized_official_credential <chr>,
#> #   basic_authorized_official_first_name <chr>,
#> #   basic_authorized_official_last_name <chr>,
#> #   basic_authorized_official_telephone_number <chr>,
#> #   basic_authorized_official_title_or_position <chr>,
#> #   basic_authorized_official_middle_name <chr>,
#> #   basic_authorized_official_name_prefix <chr>, taxonomies_code <chr>,
#> #   taxonomies_desc <chr>, taxonomies_primary <lgl>,
#> #   taxonomies_state <chr>, taxonomies_license <chr>,
#> #   taxonomies_taxonomy_group <chr>
```

Or we can flatten the whole thing and prune back later:

``` r
npi_flatten(nyc)
#> # A tibble: 44 x 46
#>       npi basic_name_pref… basic_first_name basic_last_name
#>     <int> <chr>            <chr>            <chr>          
#>  1 1.02e9 <NA>             <NA>             <NA>           
#>  2 1.02e9 <NA>             <NA>             <NA>           
#>  3 1.02e9 <NA>             <NA>             <NA>           
#>  4 1.02e9 <NA>             <NA>             <NA>           
#>  5 1.15e9 <NA>             PREMILA          MATHEWS        
#>  6 1.15e9 <NA>             PREMILA          MATHEWS        
#>  7 1.15e9 <NA>             PREMILA          MATHEWS        
#>  8 1.15e9 <NA>             PREMILA          MATHEWS        
#>  9 1.16e9 <NA>             STACEY-ANN       BROWN          
#> 10 1.16e9 <NA>             STACEY-ANN       BROWN          
#> # … with 34 more rows, and 42 more variables: basic_middle_name <chr>,
#> #   basic_credential <chr>, basic_sole_proprietor <chr>,
#> #   basic_gender <chr>, basic_enumeration_date <chr>,
#> #   basic_last_updated <chr>, basic_status <chr>, basic_name <chr>,
#> #   basic_organization_name <chr>, basic_organizational_subpart <chr>,
#> #   basic_authorized_official_credential <chr>,
#> #   basic_authorized_official_first_name <chr>,
#> #   basic_authorized_official_last_name <chr>,
#> #   basic_authorized_official_telephone_number <chr>,
#> #   basic_authorized_official_title_or_position <chr>,
#> #   basic_authorized_official_middle_name <chr>,
#> #   basic_authorized_official_name_prefix <chr>,
#> #   other_names_organization_name <chr>, other_names_code <chr>,
#> #   other_names_type <chr>, identifiers_identifier <chr>,
#> #   identifiers_code <chr>, identifiers_desc <chr>,
#> #   identifiers_state <chr>, identifiers_issuer <chr>,
#> #   taxonomies_code <chr>, taxonomies_desc <chr>,
#> #   taxonomies_primary <lgl>, taxonomies_state <chr>,
#> #   taxonomies_license <chr>, taxonomies_taxonomy_group <chr>,
#> #   addresses_country_code <chr>, addresses_country_name <chr>,
#> #   addresses_address_purpose <chr>, addresses_address_type <chr>,
#> #   addresses_address_1 <chr>, addresses_address_2 <chr>,
#> #   addresses_city <chr>, addresses_state <chr>,
#> #   addresses_postal_code <chr>, addresses_telephone_number <chr>,
#> #   addresses_fax_number <chr>
```

Now we’re ready to do whatever else we need to do with this data. Under
the hood, `npi_flatten()` has done a lot of data wrangling for us:

  - unnested the specified list columns
  - avoided potential naming collisions by prefixing the unnested names
    by their originating column name
  - joined the data together by NPI

### Validating NPIs

Use `npi_is_valid()` to check whether each element of a vector of
candidate numbers is a valid NPI number:

``` r
# Validate off NPIs
npi_is_valid(c(1234567893, 1234567898))
#> [1] TRUE
```

## Set your own user agent

By default, all request headers include a user agent that references
this repository. You can customize the user agent by setting the
`npi_user_agent` option:

``` r
options(npi_user_agent = "my_awesome_user_agent")
```

## Reporting Bugs

Did you spot a bug? I’d love to hear about it at the [issues
page](https://github.com/frankfarach/npi/issues).

## Code of Conduct

Please note that this project is released with a [Contributor Code of
Conduct](CODE_OF_CONDUCT.md). By participating in this project you agree
to abide by its terms.

## License

MIT (c) [Frank Farach](https://github.com/frankfarach)
