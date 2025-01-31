
# Core Summaries

This directory consists of a set of files to compile summaries across a set of model runs.

Most of this is written to check the existence of files and run things only if things
don't exist, so to force everything to run, clear out the scenario and across-scenario dirs.

The files contained are as follows (from the top down):

### summarizeAcrossScenarios.bat

This file sets up some environment variables such as what scenarios we will summarize,
where this code is setup, and where the combined (across scenarios) data files should reside.

It then goes through each scenario and calls [summarizeScenario.bat](summarizeScenario.bat)

After those summaries are complete, it creates a Tableau Data Extract for each summary type,
which combines the data across scenarios.  The Tableau workbook then loads data across all the
scenarios, with worksheets showing across-scenario summaries (non-white tabs) or
per-scenario summaries (with white tabs).

### summarizeScenario.bat

This script summarizes a single scenario.

(It starts with a mapping from scenario name (e.g. 2010_04_ZZZ) to some directory locations,
but this mapping will likely be elsewhere or removed.)

It works by
 * copying the files required via [copyCoreSummariesInputs.bat](copyCoreSummariesInputs.bat)
 * running the R script, [CoreSummaries.Rmd](CoreSummaries.Rmd), if needed (if any of the summary outputs don't exist)
 * converting the resulting `.rdata` files to Tableau Data Extracts via [RdataToTableauExtract.py](RdataToTableauExtract.py)
 * converting a couple of other csv files to Tablea Data Extracts via [csvToTableauExtract.py](csvToTableauExtract.py)

### copyCoreSummariesInputs.bat

This copies required files from the variously specified input directories to a single place.

If Core Summaries scripts happen as part of a model run, this will likely be unnecessary and
the summary scripts can just refer to the inputs in their original locations.

### CoreSummaries.Rmd

This R script is the bulk of the process.  It reads Trips, Tours, Persons and Households,
joining them together and joining them with skims in order to create summaries of interest.

It outputs numerious summaries both as `.csv` files and as `.rdata` files.  The
Tableau data extracts are converted from the `.rdata` files because the binary format
is more compressed and also contains information about data types.  The index columns are those
before the `freq` column.  These outputs include:

#### ActiveTransport

Active transportation summary of persons.  Sum(freq) = population

| Column Name | Description |
|-------------|-------------|
| taz | TAZ of residence |
| county_name | County of residence |
| ptype | Person type. (1:"Full-time worker"; 2:"Part-time worker"; 3:"University student"; 4:"Nonworker"; 5:"Retired"; 6:"Student of non-driving age"; 7:"Student of driving age"; 8:"Child too young for school" |
| zeroAuto | boolean, True if zero autos in the household |
| freq | Frequency of persons |
| active | Average minutes of active travel per person per weekday |
| more15 | Share of population that engages in at least 15 minutes of active travel per typical weekday |
| more30 | Share of population that engages in at least 30 minutes of active travel per typical weekday |
| wlk_trip | Share of population that makes walk trips (excluding walking as part of transit travel) |
| bik_trip | Share of population that makes bicycle trips |
| wtr_trip | Share of population that makes walk-to-transit trips |
| dtr_trip | Share of population that makes drive-to-transit trips (note these have a walk component) |
| atHomeA | Share of population that does not leave home on a typical weekday |
   
#### ActivityPattern
Activity pattern summary of persons.  Sum(freq) = population

| Column Name | Description |
|-------------|-------------|
| type | Person type string.  One of ("Full-time worker"; "Part-time worker"; "University student"; "Nonworker"; "Retired"; "Student of non-driving age"; "Student of driving age"; "Child too young for school") |
| activity_pattern | Daily activity pattern category.  One of 'H' for home, 'M' for mandatory, or 'N' for non-mandatory. |
| imf_choice | Individual mandatory tour frequency .  See https://github.com/BayAreaMetro/modeling-website/wiki/Person |
| inmf_choice | Individual non-mandatory tour frequency. See https://github.com/BayAreaMetro/modeling-website/wiki/Person |
| incQ_label | Income quartile.  One of ('Less than $30k', '$30k-$60k', '$60k-$100k', 'More than $100k') |
| freq | Frequency of persons |

#### AutomobileOwnership
Automobile Ownership summary of households.  Sum(freq) = households

| Column Name | Description |
|-------------|-------------|
| SD | Superdistrict geographical designation of residence. See https://github.com/BayAreaMetro/modeling-website/wiki/TazData |
| COUNTY | County code of residence.  See https://github.com/BayAreaMetro/modeling-website/wiki/TazData |
| county_name	| County name of residence. |
| autos | Number of autos in the household, from 0 to 4 |
| incQ	| Income quartile, from 1 to 4 |
| incQ_label | Income quartile.  One of ('Less than $30k', '$30k-$60k', '$60k-$100k', 'More than $100k') |
| walk_subzone | Walk to transit sub-zone.  One of 0,1, or 2.  See https://github.com/BayAreaMetro/modeling-website/wiki/Household |
| walk_subzone_label | String version of `walk_subzone` |
| workers | Number of workers in the household |
| kidsNoDr| Boolean; True iff the household has children in the househole that don't drive
 (either pre-school age or school age) |
| freq | Frequency of households |

#### AutoTripsVMT_perOrigDestHomeWork
Automobile trips for VMT summing.  Sum(trips) = total auto trips for an average weekday.

| Column Name | Description |
|-------------|-------------|
| orig_taz | Origin TAZ for the trip |
| dest_taz | Destination TAZ for the trip |
| taz | TAZ of residence for the tripmaker |
| WorkLocation | TAZ of work location for the tripmaker, or 0 if none |
| vmt | VMT for the trips, a sum of `vmt_indiv` and `vmt_joint` |
| vmt_indiv	| VMT from individual trips |
| vmt_joint | VMT from joint trips |
| trips | Number of (person) trips |
| vehicle_trips | Number of vehicle trips |

#### AutoTripsVMT_personsHomeWork
Automobile trips by person.  Sum(freq) = population

| Column Name | Description |
|-------------|-------------|
| COUNTY | County code of residence |
| county_name	| County name of residence |
| taz	| TAZ of residence |
| WorkLocation	| TAZ of Work location or 0 if none |
| freq | Number of persons with this home/work combination |

#### CommuteByEmploymentLocation
Commute characteristics by employment location.  Sum(freq) = commute tours

| Column Name | Description |
|-------------|-------------|
| dest_COUNTY | County code of work commute destination |
| dest_county_name | County name of work commute destination |
| dest_SD	| Superdistrict of work commute destination |
| tour_mode	| Work tour mode.  See https://github.com/BayAreaMetro/modeling-website/wiki/IndividualTour |
| freq | Number of commute tours |
| totCost	| Total round-trip out of pocket costs in $2000.  Sum of `cost` and `parking_cost`. |
| cost | See https://github.com/BayAreaMetro/modeling-website/wiki/SimpleSkims#cost-skims for what's included |
| parking_cost | Parking costs |
| distance | Distance of commute |
| cost_fail | Commute tours for which the cost lookup failed |

#### CommuteByIncomeHousehold
Commute characteristics by household location. Sum(freq) = commute tours

| Column Name | Description |
|-------------|-------------|
| res_COUNTY | County code of residence |
| res_county_name | County name of residence |
| res_SD | Superdistrict of residence |
| orig_taz | Origin TAZ |
| tour_mode | Work tour mode.  See https://github.com/BayAreaMetro/modeling-website/wiki/IndividualTour |
| incQ	| Income quartile |
| incQ_label | Income quartile.  One of ('Less than $30k', '$30k-$60k', '$60k-$100k', 'More than $100k') |
| freq | Number of commute tours.
| totCost  | Total round-trip out of pocket costs in $2000.  Sum of `cost` and `parking_cost`. |
| cost | See https://github.com/BayAreaMetro/modeling-website/wiki/SimpleSkims#cost-skims for what's included |
| parking_cost | Parking costs |
| distance | Distance of commute |
| duration	| Duration of commute |
| cost_fail | Commute tours for which the cost lookup failed |
| time_fail | Commute tours for which the time lookup failed |

#### CommuteByIncomeJob
Commute characteristics by job location.  Sum(freq) = commute tours

Columns are the same as CommuteByIncomeHousehold, except they are summed 
to job/destination geographies instead of to residential geographies.

#### JourneyToWork
Workplace location summaries (including when tours are not made.)  Sum(freq) = persons with work locations

| Column Name | Description |
|-------------|-------------|
| homeCOUNTY | County code of residence |
| home_county_name	| County name of residence |
| HomeTAZ | TAZ of residence |
| WorkLocation | TAZ of work location |
| workCOUNTY | County code of work location |
| work_county_name | County name of work location |
| freq | Number of persons with home/work locations as described |
| Income | Average income |

#### PerTripTravelTime
Sum(freq) = trips

| Column Name | Description |
|-------------|-------------|
| incQ | Income quartile |
| incQ_label | Income quartile.  One of ('Less than $30k', '$30k-$60k', '$60k-$100k', 'More than $100k') |
| trip_mode | Trip mode. See https://github.com/BayAreaMetro/modeling-website/wiki/IndividualTrip |
| tour_purpose | Tour purpose for the trip.  See https://github.com/BayAreaMetro/modeling-website/wiki/IndividualTour |
| freq | Number of trips |
| num_participants | Number of participants for these trips |
| trvlTime | Average travel time |
| time_fail | Commute tours for which the time lookup failed |

#### TimeOfDay
Sum(freq) = tours

| Column Name | Description |
|-------------|-------------|
| SD | Superdistrict of residence |
| COUNTY | County of residence |
| county_name | County name of residence |
| simple_purpose | Simple tour purpose, one of ('work', 'school', 'non-work', 'at-work', 'college') |
| tour_mode | Tour mode.  See https://github.com/BayAreaMetro/modeling-website/wiki/IndividualTour |
| start_hour | Start hour for the tour, in [5,23] |
| end_hour | End  hour for the tour, in [5,23] |
| freq | Number of tours |
| num_participants | Number of person participants |

#### TimeOfDay_personsTouring
Summary of how many persons are touring at a given hour.

| Column Name | Description |
|-------------|-------------|
| simple_purpose | Simple tour purpose, one of ('work', 'school', 'non-work', 'at-work', 'college') |
| persons_touring | Number of persons touring |
| hour | The hour of their tour, in [5,23] |

#### TravelCost
Travel costs by household.  Sum(freq) = households

| Column Name | Description |
|-------------|-------------|
| SD | Superdistrict of residence |
| COUNTY | County code of residence |
| county_name | County name of residence |
| people | Persons per household, capped at 6 |
| incQ | Income Quartile |
| incQ_label | Income quartile.  One of ('Less than $30k', '$30k-$60k', '$60k-$100k', 'More than $100k') |
| autos | Number of autos in the household, from 0 to 4 |
| freq | Number of households |
| total_cost | Total travel costs |
| trip_cost_indiv | Total travel costs from individual trips |
| trip_cost_joint	| Total travel costs from joint trips |
| cost_fail | Trips for which the cost lookup failed |
| pcost_indiv | Parking cost from individual trips |
| pcost_joint | Parking cost from joint trips |


#### TripDistance
Distance summaries for trips.  Sum(freq) = trips

| Column Name | Description |
|-------------|-------------|
| autoSuff | Auto sufficiency code, one of [0,1,2] |
| autoSuff_label | Auto sufficiency label, one of ('Zero automobiles','Automobiles < workers','Automobiles >= workers') |
| incQ | Income Quartile |
| incQ_label | Income quartile.  One of ('Less than $30k', '$30k-$60k', '$60k-$100k', 'More than $100k') |
| timeCode | Time period of trip, one of ('AM','MD','PM','EV','EA') |
| timeperiod_label | Time period label, one of ('AM Peak', 'Midday', 'PM Peak', 'Evening', 'Early AM') |
| trip_mode | Trip mode. See https://github.com/BayAreaMetro/modeling-website/wiki/IndividualTrip |
| tour_purpose | Tour purpose for the trip.  See https://github.com/BayAreaMetro/modeling-website/wiki/IndividualTour |
| freq | Number of trips |
| distance | Total trip distance |

#### VehicleMilesTraveled
Auto vehicle miles traveled summed to persons. Sum(freq) = population

| Column Name | Description |
|-------------|-------------|
| COUNTY | County code of residence |
| county_name | County name of residence |
| SD | Superdistrict of residence |
| taz | TAZ of residence |
| walk_subzone | Walk to transit sub-zone.  One of 0,1, or 2.  See https://github.com/BayAreaMetro/modeling-website/wiki/Household |
| walk_subzone_label | String version of `walk_subzone` |
| ptype | Person type. (1:"Full-time worker"; 2:"Part-time worker"; 3:"University student"; 4:"Nonworker"; 5:"Retired"; 6:"Student of non-driving age"; 7:"Student of driving age"; 8:"Child too young for school" |
| ptype_label	| Person type label from 'ptype' |
| autoSuff | Auto sufficiency code, one of [0,1,2] |
| autoSuff_label | Auto sufficiency label, one of ('Zero automobiles','Automobiles < workers','Automobiles >= workers') |
| freq | Number of persons |
| vmt_indiv | Mean VMT from individual trips |
| vmt_joint	| Mean VMT from joing trips |
| vmt | Mean VMT |
| person_trips | Total person Trips |
| vehicle_trips | Total vehicle Trips |

### VehicleMilesTraveled_households

Auto vehicle miles traveled summed to households.
Same columns as VehicleMilesTraveled but Sum(freq) = households and no ptype columns.

#### Additional Output Tables
It also outputs updated `.rdata` versions of the Trip, Tours, Persons and Households table, 
with the extra data fields added.
