Lab 2: Map Census data and create an R Markdown document
================
Elisabeth Ericson
17 Sep 2021

I have no idea why it is displaying author and date on the same line.
Tried Googling it, phrased a variety of ways, and only came up with
unrelated issues. That said, here is the assignment…

### import libraries

``` r
library(tidyverse)
library(tidycensus)
library(sf)
```

### view 2019 ACS variables

commented out because it was unnecessary after the first time

``` r
# vars <- load_variables(2019, "acs5")
# view(vars)
```

### define ACS variables to select

``` r
acs_vars <- c("B01001_001", # Total population estimate
              "B05002_013", # Foreign born population
              "B05002_003") # Population born in state of residence (Pennsylvania)
```

### define Census tracts for Center City West

obtained by cross-referencing Census reference maps with Center City
boundaries listed on Wikipedia, plus some digging in the data table
because the full ID was not clear from the reference maps

``` r
myTracts <- c("42101000300",
               "42101000401",
               "42101000402",
               "42101000700",
               "42101000801",
               "42101000803",
               "42101000804",
               "42101001201",
               "42101001202")
```

### import and clean up Census variables

``` r
acsTractsCCW.2019.sf <- get_acs(geography = "tract",
                             year = 2019,
                             variables = acs_vars,
                             geometry = TRUE,
                             state = "PA",
                             county = "Philadelphia",
                             output = "wide") %>%
  dplyr::select(GEOID, NAME, all_of(paste0(acs_vars,"E"))) %>%
  rename(totalpop.2019 = B01001_001E,
         foreignborn.2019 = B05002_013E,
         borninstate.2019 = B05002_003E) %>%
  mutate(pctforeignborn.2019 = foreignborn.2019/totalpop.2019*100,
         pctborninstate.2019 = borninstate.2019/totalpop.2019*100,
         Neighborhood = ifelse(GEOID %in% myTracts,
                               "Center City West",
                               "Rest of Philadelphia"))
```

### project to WGS84

``` r
acsTractsCCW.2019.sf <- acsTractsCCW.2019.sf %>%
  st_transform(crs = "EPSG:4326")
```

### plot data on map

``` r
ggplot()+
  geom_sf(data = acsTractsCCW.2019.sf, aes(fill = pctforeignborn.2019),
          color = "transparent")+
  geom_sf(data = acsTractsCCW.2019.sf %>%
            filter(Neighborhood == "Center City West") %>%
            st_union(),
          color = "white",
          fill = "transparent")+
  scale_fill_viridis_c('Percent foreign-born',
                     direction = -1,
                     option = 'B',
                     alpha = 0.6)+
  labs(title = "Foreign-born population",
       subtitle = "in Center City West compared to rest of Philadelphia",
       caption = "Data: US Census Bureau, ACS 5-year estimates")
```

![](lab2_ee_files/figure-gfm/unnamed-chunk-7-1.png)<!-- -->
