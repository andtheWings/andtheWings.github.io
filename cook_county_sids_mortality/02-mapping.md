# (PART) Exploration {-}

# Mapping SIDS-related Deaths


```
#> PhantomJS not found. You can install it with webshot::install_phantomjs(). If it is installed, please make sure the phantomjs executable can be found via the PATH variable.
```

<iframe src="http://danielriggins.com/widgets/cook_county_sids_deaths.html" width="100%" height="400px" data-external="1"></iframe>

*Visit [http://danielriggins.com/widgets/cook_county_sids_deaths.html](http://danielriggins.com/widgets/cook_county_sids_deaths.html) for a full-screen view.*

## Code to produce the map

### Load Dependencies


```r
box::use(
    dplyr[
        case_when,
        full_join,
        mutate,
        select
    ],
    leaflet[
        addLayersControl,
        addLegend,
        addPolygons,
        addProviderTiles, 
        leaflet, 
        setMaxBounds, 
        setView
    ],
    leaflet.extras[addFullscreenControl],
    magrittr[`%>%`],
    sf[...],
    tibble[view]
)
```

### Reshape data for use in the map


```r
# Load SIDS death data
df <-
    # Load cached geospatial features
    readRDS("data/coords_and_pop_est.RDS") %>%
    # Join to cached dataframe
    full_join(readRDS("data/df.RDS")) %>%
    # Select ID and outcome variables
    select(fips, count_asphyxia) %>%
    # Turn outcome into an ordinal factor
    mutate(
        death_count = factor(
            case_when(
                count_asphyxia == 0 ~ "No Deaths",
                count_asphyxia == 1 ~ "One Death",
                count_asphyxia == 2 ~ "Two Deaths",
                count_asphyxia == 3 ~ "Three Deaths",
                count_asphyxia == 4 ~ "Four Deaths",
                count_asphyxia == 5 ~ "Five Deaths",
                count_asphyxia == 6 ~ "Six Deaths"
            ),
            ordered = TRUE,
            levels = c(
                "No Deaths", 
                "One Death", 
                "Two Deaths", 
                "Three Deaths", 
                "Four Deaths", 
                "Five Deaths", 
                "Six Deaths"
            )
        )
    )

# Configure color palette
sids_palette <- 
    leaflet::colorFactor(
        palette = "magma",
        reverse = TRUE,
        levels = c(
                "No Deaths", 
                "One Death", 
                "Two Deaths", 
                "Three Deaths", 
                "Four Deaths", 
                "Five Deaths", 
                "Six Deaths"
            )
    )

# Create map widget object
m <- leaflet(df) %>%
    # Use CartoDB's background tiles
    addProviderTiles("CartoDB.Positron") %>%
    # Center and zoom the map to Cook County
    setView(lat = 41.816544, lng = -87.749500, zoom = 9) %>%
    # Add button to enable fullscreen map
    addFullscreenControl() %>%
    # Add census tract polygons colored to reflect the number of deaths
    addPolygons(
        # No borders to the polygons, just fill
        stroke = FALSE,
        # Color according to palette above
        color = ~ sids_palette(death_count),
        # Group polygons by number of deaths for use in the layer control
        group = ~ death_count,
        # Make slightly transparent
        fillOpacity = 0.7,
        # Click on the polygon to get its ID
        popup = ~ paste0("<b>FIPS ID:</b> ", as.character(fips))
    ) %>%
    #Add legend
    addLegend(
        title = "Number of SIDS deaths <br> per census tract",
        values = ~ death_count,
        pal = sids_palette,
        position = "topright"
    ) %>%
    # Add ability to toggle each factor grouping on or off the map
    addLayersControl(overlayGroups = c(
                "No Deaths", 
                "One Death", 
                "Two Deaths", 
                "Three Deaths", 
                "Four Deaths", 
                "Five Deaths", 
                "Six Deaths"
            ),
            position = "topleft"
        )
```
