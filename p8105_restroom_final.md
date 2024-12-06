p8105_restroom_final
================

### The group members (names and UNIs)

- Carrie Wu, Kw3104
- Huiyan Ni, Hn2453
- Rita Wang, Ryw2109
- Megan Panier, Map2365
- Minghe Wang, mw3845

### Operational Restrooms Across NYC

Although public restrooms are readily available to us, they are not
always usable. This could be due to malfunctions of the restroom or the
entirety of the restroom is closed due to the location being closed for
holiday or for the season. Analyzing the available of operational
restrooms in NYC may help to prevent individuals from going to public
restroom while worrying it may not be available.

#### The intended final products

Our final product will be a comprehensive webpage with four main
sections: an introduction to the project and dataset, data
visualizations comparing restroom conditions, an interactive map of
operational restrooms, and a detailed report with findings and future
recommendations. This will allow users to explore restroom accessibility
and discover patterns in restroom availability across different areas of
NYC.

#### The anticipated data sources and planned analyses / visualizations / coding challenges

We will source our data from NYC Open Data on Public Restrooms and the
train station dataset from our class resources. Planned analyses will
include mapping operational restrooms by location using latitude and
longitude, visualizing accessibility across NYC train stations, and
comparing restrooms by accessibility and gender accommodations. A
significant challenge may be aligning columns from the two datasets for
accurate analysis.

#### The planned timeline

Our timeline includes completing data tidying by 11/22/2024,
visualizations by 11/30/2024, the webpage by 12/06/2024, and a recorded
presentation by 12/12/2024. Our team will collaborate on GitHub for
proposal creation, using .Rmd to render the document as a GitHub
document for submission.

### Data Cleaning

``` r
# Import data
restroom_df = read_csv(here::here("./Data/Public_Restrooms_20241203.csv")) %>% 
  janitor::clean_names()
```

    ## Rows: 1047 Columns: 14
    ## ── Column specification ────────────────────────────────────────────────────────
    ## Delimiter: ","
    ## chr (12): Facility Name, Location Type, Operator, Status, Open, Hours of Ope...
    ## dbl  (2): Latitude, Longitude
    ## 
    ## ℹ Use `spec()` to retrieve the full column specification for this data.
    ## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

``` r
subway_df = read_csv(here::here("./Data/NYC_Transit_Subway_Entrance_And_Exit_Data.csv")) %>% 
  janitor::clean_names()
```

    ## Rows: 1868 Columns: 32
    ## ── Column specification ────────────────────────────────────────────────────────
    ## Delimiter: ","
    ## chr (22): Division, Line, Station Name, Route1, Route2, Route3, Route4, Rout...
    ## dbl  (8): Station Latitude, Station Longitude, Route8, Route9, Route10, Rout...
    ## lgl  (2): ADA, Free Crossover
    ## 
    ## ℹ Use `spec()` to retrieve the full column specification for this data.
    ## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

``` r
# Clean restroom data
restroom_df = restroom_df %>% 
  select(
    -website, -operator, -hours_of_operation 
  ) %>% 
  rename(
    restroom_latitude = latitude,
    restroom_longitude = longitude,
    restroom_location = location
  ) %>% 
  mutate(
    rest_room_latitude = as.numeric(restroom_latitude),
    rest_room_longitude = as.numeric(restroom_longitude),
    restroom_location = st_as_sfc(restroom_location), #convert to point
    open = factor(
      open,
      levels = c("Future", "Seasonal", "Year Round"),
      ordered = TRUE
    ),
    accessibility = factor(
      accessibility,
      levels = c("Not Accessible", "Partially Accessible", "Fully Accessible"),
      ordered = TRUE
    ),
    changing_stations = case_when(
      changing_stations %in% c("Yes, in single-stall all gender restroom only",
                                "Yes, in women's restroom only",
                                "Yes") ~ 1,
      changing_stations == "No" ~ 0
    ),
    status = case_when(
      status %in% c("Operational",
                    "Closed for Construction",
                    "Closed") ~ 1,
      status == "Not Operational" ~ 0
    )
  ) 

# Convert dataframe to sf for spatial operations
restroom_sf = st_sf(restroom_df, crs = 4326)

restroom_near_transit = restroom_df %>% 
  filter(location_type == 'Transit')
```

MTA reports that there is 63 out of 423 subway stations provide
restrooms 7am - 7pm and in restroom_df that we imported from NYC Open
Data, only 5 restroom were marked as in the location of subway out of
1047 recorded restrooms.
