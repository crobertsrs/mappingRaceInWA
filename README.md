# mappingRaceInWA
Using census data to map race in various ways in mostly King County in WA state.

## General Process

1. Find census data using the [American Fact Finder](https://factfinder.census.gov) to be saved in the data folder. It should have:
    - Race (preferrably the mosst disaggregated possible)
    - Some location level (Tract for Seattle which will later be recut, Census place for the rest of WA)
    - Sets for 2000 and 2010 and maybe 2017

2. Find map files for the same years and data to be saved in the maps/census folder. They are almost certainly [Census TIGER files](https://www.census.gov/geo/maps-data/data/tiger-line.html) created in 2010 for 2000 and 2010 years

3. Check for differences in map files. Places and tracts change over time, so either make an assumption of consistency or a plan for unifying the data.
    - Seattle area is getting recut to a common neighborhood map, so those need to be calculated. We will assume that the GIS geometry cutting process is sound and no assumptions need to be made comparing the resulting 2000 map with the resulting 2010 map.
    - The rest of King County is using places which definitely have some boundary changes and renaming across years, but my audience is likely already using data grouped by these features, so it would be nice if they were right. 

4. Create concordance tables for 2000 Seattle tracts to Seattle neighborhoods and 2010 Seattle tracts to Seattle neighborhoods.

5. Create concordance tables for King County places between 2000 and places in 2010. 


## Actual Process

1. Found tables from the census that have all race combinations, Place, Tract, and Zip

2. Renamed columns and separated the three data sets at first just into one for Place. Used the following excel code to look in BV1 for each race category and concat with a short code and slash, remove the trailling slash, and leave blank of no match:
2000
```
=IFERROR(LEFT(CONCAT(IFERROR(IF(FIND("White",BV1),"WH/",""),""),IFERROR(IF(FIND("Black",BV1),"AA/",""),""),IFERROR(IF(FIND("American Indian",BV1),"AI/",""),""),IFERROR(IF(FIND("Asian",BV1),"AS/",""),""),IFERROR(IF(FIND("Native Hawaiian",BV1),"PI/",""),""),IFERROR(IF(FIND("Some other",BV1),"OT/",""),"")),LEN(CONCAT(IFERROR(IF(FIND("White",BV1),"WH/",""),""),IFERROR(IF(FIND("Black",BV1),"AA/",""),""),IFERROR(IF(FIND("American Indian",BV1),"AI/",""),""),IFERROR(IF(FIND("Asian",BV1),"AS/",""),""),IFERROR(IF(FIND("Native Hawaiian",BV1),"PI/",""),""),IFERROR(IF(FIND("Some other",BV1),"OT/",""),"")))-1),"")
```
2010
```
=IFERROR(LEFT(CONCAT(IFERROR(IF(FIND("White",BV1),"WH/",""),""),IFERROR(IF(FIND("Black",BV1),"AA/",""),""),IFERROR(IF(FIND("American Indian",BV1),"AI/",""),""),IFERROR(IF(FIND("Asian",BV1),"AS/",""),""),IFERROR(IF(FIND("Native Hawaiian",BV1),"PI/",""),""),IFERROR(IF(FIND("Some Other",BV1),"OT/",""),"")),LEN(CONCAT(IFERROR(IF(FIND("White",BV1),"WH/",""),""),IFERROR(IF(FIND("Black",BV1),"AA/",""),""),IFERROR(IF(FIND("American Indian",BV1),"AI/",""),""),IFERROR(IF(FIND("Asian",BV1),"AS/",""),""),IFERROR(IF(FIND("Native Hawaiian",BV1),"PI/",""),""),IFERROR(IF(FIND("Some Other",BV1),"OT/",""),"")))-1),"")
```

3. Checked to see which GEOIDs matched and how many were missing. 
The following places appear in 2000 but not in 2010 | What I found in Geography notes | What I did:
Cascade-Fairwood CDP, Washington        | Part annexed by Renton and part new Fairwood    | Divided into 2 and gave half to Renton and half to Fairwood (added)
Lea Hill CDP, Washington                | Deleted and annexed by Auburn city              | Added to Auburn City
Riverton-Boulevard Park CDP, Washington | Split into Riverton and Boulevard Park          | Divided into 2 and gave half to Burien and half to Boulevard Park (added)
West Lake Sammamish CDP, Washington     | Annexed by Bellevue City                        | Added to Bellevue City
The following places appear in 2010 but not in 2000:
Boulevard Park CDP, Washington          | Made from split of Riverton-Boulevard Park CDP  | No change needed
Fairwood CDP (King County), Washington  | Formed from part of Cascade-Fairwood CDP        | No change needed
Klahanie CDP, Washington                | New                                             | Did not include
Lake Holm CDP, Washington               | New                                             | Did not include
Riverton CDP, Washington                | Made from split of Riverton-Boulevard Park CDP  | No change needed
Shadow Lake CDP, Washington             | New                                             | Did not include
Wilderness Rim CDP, Washington          | New                                             | Did not include
The following places appear in 2017 but not in 2010:
Riverton CDP, Washington                | Made from split of Riverton-Boulevard Park CDP  | See 2000, annexed by Burien


4. Made all the cosmetic changes needed to get tables ready, split into a smaller file and matched 2000 to 2010 based on the above.

5. At this point realized I should try to grab data from 2017, so I went back and found those, realizing that the data showing place of birth by race does not allow for detailed multiracial categories. 

6. Census place of birth is broken up into a lot of caegories including US citizens who are born outside the 50 states or in other countries. Some of the data we are interested in focuses specifically on generational Americans. For each Place of Birth by Race by year table, I added the columns for born in the state of the geography and born in another US state to be the "Native Born" category and subtracted that from the total to get the "Foreign Born" category.

7. I made the Place concordances by hand by creating a new sheet using the 2017 places, but looking up the 2000 data and editing the lookup equations when necessary.

8. I used QGIS to form a union of the Seattle Census tracts and Seattle Neighborhoods, deleted any of the areas that did not overlap, then calculated the areas of both the original tracts and the neighborhoods. In the spreadsheet, I used the areas to calculate the relative value of each statistic for ever tract/neighborhood overlap, then created a pivot table to add up the values by neighborhood. Those neighborhoods fed into the table.

7. The result is a table with one row per area (neighborhoods for Seattle and Census Place for the rest of King County) with columns for each datapoint per year (e.g. 2000 PoB Black/African American, 2000 Asian...2017 Black/African American, etc.)