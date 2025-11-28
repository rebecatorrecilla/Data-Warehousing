# Data-Warehousing
## Statement
Given the ACME-Flying Use Case, this project aims to design a Data Warehouse that allows perform descriptive analysis based in three parts:
1. Creation of a multidimensional database design that allows a correct execution of the queries. To draw the multidimensional schema the graphical editor draw.io was used.
2. Develop an ETL process that allows to extract, transform and load the data from existing databases to the developed database in the previous step with the `pygramtel` Python library.
3. Compare the execution time of the queries over the data sources and over the multidimensional schemas in the data warehouse implemented in DuckDB. 

## Multidimensional Design 
For this section, a multidimensional schema was created in order to easily retrieve KPIs about aircraft utilization (namely `FH`, `TO`, `ADIS`, `ADOS`, `ADOSS`, `ADOSU`, `DU`, `DC`, `DYR`, `CNR`, `TDR` and `ADD`) as well as the logbook reporting (namely `RRh`, `RRc`, `PRRh`, `PRRc`, `MRRh` and `MRRc`).

All these metrics were obtained from the datasets `AIMS` and `AMOS` while also being complemented by the following sources:
* `aircraft-manufacturerinfo-lookup`: a file containing for every aircraft registration code, its manufacturer registration code, the aircraft model and manufacturer.
* `maintenance_personnel`: another file containing the maintenance personnel, their identifier and the code of the airport where they work (any other employee not in this file is assumed to be a pilot).

The design was implemented with the main objetive of being able to retrieve the KPIs based on their granularity (daily, weekly...) while grouping the metrics by model, manufacturer, aircraft or reporting person depending on the metric. 

## Extract-Transform-Load (ETL) Process Design
In the second part we created an Extract-Transform-Load (ETL) process that can be executed in order to **extract** data from the `AIMS` and `AMOS` operational databases and additionally provided data sources, **transform** these data to conform to the multidimensional schema previously defined and **load** the data into the schema. 

The design ETL process followed the next steps:

#### Extraction
* We connected to the source databases `AIMS` and `AMOS` for extracting the raw operational data.
* We also extracted the data from the additional data sources (`aircraft-manufacturerinfo-lookup.csv` and `maintenance_personnel.csv`)

#### Transformation
- We integrated the data coming from `AIMS` and `AMOS` data sources based on their shared attributes.
- Next, we complemented the operational data coming from `AIMS` and `AMOS` by means of performing a lookup to the external data sources about
   - *Aircraft manufacturer information* (`aircraft-manufacturerinfo-lookup.csv`)such that with each aircraft registration code, the ETL also provides its manufacturer registration code, the aircraft model and manufacturer.
   - *Maintenance personnel employement place* (`maintenance_personnel.csv`) such that for each person from the maintenance personnel the ETL also provides information at which airport this person works.
- We derived additional attributes, by means of, but not limited to value convserion and formula calculation, in order to enable the calculation of the requested KPIs.
- Lastly, we also improved the quality of the data sources by checking and fixing only the following three business rules:
   - In a Flight, actualArrival is porterior to actualDeparture, related to BR-23.
   - Two non-cancelled Flights of the same aircraft cannot overlap, related to BR-21.
   - The aircraft registration in a post flight report must be an aircraft.
 
#### Loading
* This part consisted in the creation of the DW in a DuckDB database.
* We loaded the dimension tables of the multidimensional schema, paying special attention to keep the different aggregation levels.
* Finally, we loaded the fact tables of our multidimensional schema, allowing the calculation of all the metrics needed to generate the required KPIs.

## Comparison of the queries over the Data Warehouse and the sources.
In the last section we focused on checking the queries over the raw data in PostgreSQL and reimplement them over the DW in DuckDB using the `query__test.py` file. 









