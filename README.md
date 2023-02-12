# phac-vaccination-coverage

Source: Public Health Agency of Canada - [COVID-19 vaccination in Canada](https://health-infobase.canada.ca/covid-19/vaccination-coverage/): <https://health-infobase.canada.ca/covid-19/vaccination-coverage>. 

Direct link to data: <https://health-infobase.canada.ca/src/data/covidLive/vaccination-coverage-byAgeAndSex-overTimeDownload.csv>
Excessed: 2023-02-01

Contains: 

- Vaccine administration - by province, dose, age, gender
- Also (!) contains popuplation - by province, dose, age, gender

Notes:

- Frequency: weekly
- Updated: weekly (prior to Fall 2022), monthly (since Fall 2022)

Variables and Fields:
- values: Data are recorded as totals since 2020. Hence to obtain weekly numbers of doses adminstered, data between consecuive weeks needs to be substracted from one another (as show in code below)
- dose type: The names for the doses (first, full,additionl 2nd additional) has been changing over time, but
- Age field contains irrelevant  columns (likely started at some point of time then abondoned - need to be removed as noise)
- Sex field also contains relevant  columns (various other non-binary genders that have not been populate yet - as population data is not available for these


Related datasets - from the same page:

```
# Figure 1
url <- "https://health-infobase.canada.ca/src/data/covidLive/vaccination-coverage-map.csv"
# Figure 5
url <- "https://health-infobase.canada.ca/src/data/covidLive/vaccination-coverage-byVaccineType.csv"
```


###  Code to cache and clean data

```
# Figure 3
library(data.table) 
phac.csv <- "vaccination-coverage-byAgeAndSex-overTimeDownload.csv"
url <- paste0("https://health-infobase.canada.ca/src/data/covidLive/",phac.csv)
dt  <- fread(url)
write.csv(dt, paste0(phac.csv, "-", dateToday,".csv"))


dt <- dt[ age %ni% c("Not reported", "Unknown", "0–15", "16–69","70–74" , "75–79", "18–69","0–17" )]
dt <- dt[sex=="All sexes"]; dt$sex <- NULL

cols <-  dt %>% names %wo% c("prename"  , "week_end"  ,  "sex"   , "age" );cols
cols.weekly <- paste0(cols, ".weekly")
dt [, (cols):= lapply(.SD, as.numeric), .SDcols = cols]
setorder(dt, week_end     ) # to do shift
dt [, (cols.weekly):= lapply(.SD, function(x) {x-shift(x,1) }), .SDcols = cols, by=.(prename, age ) ]


```

