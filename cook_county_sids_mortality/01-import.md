# Import the Data

## Dependencies


```r
# Load needed modules
box::use(
    dplyr[full_join, glimpse, select],
    janitor[clean_names],
    magrittr[`%>%`],
    readxl[read_xlsx],
    sf[st_set_geometry],
    sids_data_wrangling = ./modules/sids_data_wrangling,
    tibble[as_tibble]
)
```

## Initial import

First, read in the excel file that was originally shared for this project:


```r
raw <- 
    # Parse excel file
    read_xlsx("data/finaldataforanalysis3_220121.xlsx") %>%
    # Clean up variability in naming conventions
    clean_names()
```

## Join census tract populations

Then, import census population data:


```r
## This is a custom function I wrote that 
## pulls data from the TidyCensus API about
## the population count of people under five 
## years old and about spatial features 
## for each census tract. I have commented it 
## out and saved the result in an RDS file
## so as to not make a new call to the API 
## every time this script is run. You can
## inspect the function definition in the 
## modules folder of the source code.

# coords_and_pop_est <- 
#     sids_data_wrangling$get_coords_and_pop_est(raw)
# 
# saveRDS(coords_and_pop_est, "data/coords_and_pop_est.RDS")

coords_and_pop_est <- readRDS("data/coords_and_pop_est.RDS")

# Join the population counts to the imported dataframe
df <- 
    coords_and_pop_est %>%
    # Drop geospatial features
    st_set_geometry(NULL) %>%
    # Convert to tibble format
    as_tibble() %>%
    # And join to raw
    full_join(raw)
#> Joining, by = "fips"

# Preview the data
glimpse(df)
#> Rows: 1,315
#> Columns: 32
#> $ fips                                 <dbl> 17031807500, …
#> $ pop_under_five                       <dbl> 151, 192, 21,…
#> $ count_asphyxia                       <dbl> 0, 0, 1, 0, 0…
#> $ count_opioid_death                   <dbl> 1, 7, 2, 2, 6…
#> $ svi_socioeconomic                    <dbl> 0.1269, 0.593…
#> $ svi_household_composition_disability <dbl> 0.1728, 0.803…
#> $ svi_minority_language                <dbl> 0.7024, 0.677…
#> $ svi_housing_transportation           <dbl> 0.3690, 0.528…
#> $ svi_summary_ranking                  <dbl> 0.2470, 0.679…
#> $ pe_foreignborn                       <dbl> 31.6, 2.0, 1.…
#> $ pe_marriedmales                      <dbl> 62.5, 23.0, 3…
#> $ pe_marriedfemales                    <dbl> 56.6, 23.0, 2…
#> $ pedivorcewidowedmale                 <dbl> 6.4, 16.9, 7.…
#> $ pedivorcewidowedfemale               <dbl> 16.8, 34.7, 3…
#> $ pelessthanhighschool                 <dbl> 7.1, 9.2, 8.0…
#> $ highschooldiploma                    <dbl> 14.6, 28.4, 2…
#> $ somecollege                          <dbl> 12.8, 26.4, 3…
#> $ collegediploma                       <dbl> 65.5, 36.0, 3…
#> $ black                                <dbl> 2.5, 97.4, 96…
#> $ white                                <dbl> 58.3, 0.7, 1.…
#> $ hispanic                             <dbl> 5.6, 0.0, 2.2…
#> $ male                                 <dbl> 48.8, 50.8, 3…
#> $ percent_enployed                     <dbl> 61.6, 49.0, 4…
#> $ incomelt10                           <dbl> 0.0, 15.7, 10…
#> $ incomelt25                           <dbl> 3.6, 15.6, 22…
#> $ incomelt50                           <dbl> 10.9, 15.9, 2…
#> $ incomelt75                           <dbl> 15.7, 27.6, 1…
#> $ incomegt75                           <dbl> 69.8, 25.3, 2…
#> $ privateinsurance                     <dbl> 78.9, 55.5, 5…
#> $ publicinsurance                      <dbl> 26.4, 43.5, 5…
#> $ noinsurance                          <dbl> 2.8, 12.2, 13…
#> $ spanish_language                     <dbl> 6.0, 2.1, 0.7…
```

## Save for use in other chapters


```r
saveRDS(df, file = "data/df.RDS")
```

