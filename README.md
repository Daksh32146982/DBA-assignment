# DBA-assignment

Databases and Analytics coursework, University of West London, 2025/26 Semester 2.

Three Google Colab notebooks are contained in this repository for the NorthStar Urban Mobility and Logistics case study. It includes SQL inside R, R analytics, Python data processing, and optimising queries and database design for MongoDB Atlas.


## About the case study

NorthStar Urban Mobility and Logistics is a UK regional operation providing shuttle services, last mile delivery, warehouses, an EV charging network and a mobile platform for customers and drivers. The firm has expanded through several smaller operators and its internal data is now housed in older systems which are not communicating properly.

The management has witnessed declining performance amidst increasing demand. Delays are on the rise, complaints are rising, vehicle downtime is growing and in certain zones the performance is a constant struggle. The four directors of the board all point to different problems – operations to "hubs and routes," customer experience to "disconnected reporting," finance to "unprofitable contracts," and technology to "the database structure.

The notebooks in this repository examine each of these theories in light of the actual data, determine where the real issues lie, and suggest a hybrid relational/NoSQL design that will solve these issues.

## What is in this repository

- `data/` folder with the nine raw CSV files plus the data dictionary
- Three notebooks:
  - `DBA_Notebook1_SQL_R_Analysis_.ipynb` - SQL in R and R analytics
  - `DBA_Notebook2_Python_Analysis.ipynb` - Python data processing
  - `DBA_Notebook3_MongoDB_.ipynb` - MongoDB Atlas and query optimisation
- `README.md` - this file
- `.gitignore` - standard Python gitignore


## The dataset

The data folder contains all nine CSVs from the case study and a data dictionary. The record counts are:

- `hubs.csv` - 8 hubs
- `vehicles.csv` - 120 vehicles
- `drivers.csv` - 170 drivers
- `customers.csv` - 650 customers
- `orders.csv` - 1,250 orders
- `deliveries.csv` - 950 deliveries
- `incidents.csv` - 280 incidents
- `complaints.csv` - 320 complaints
- `app_events.csv` - 640 app events
- `data_dictionary.csv` - schema reference

The README provided with the data indicated the presence of inconsistent categorical data, missing data, and unknown delay and failure patterns. All three are present and are dealt with in the notebooks. In particular, there are seven real zones on the zone columns distributed over sixteen spellings that are fixed in the cleaning step.


## How the notebooks get the data

Anyone with the link can open a notebook in Colab and run it end-to-end, without uploading anything since the three notebooks load the CSV files directly from this GitHub repository. The data urls have this format:

```
https://raw.githubusercontent.com/Daksh32146982/DBA-assignment/refs/heads/main/data/orders.csv
```

Each notebook features a flag at the top of the setup section that will switch from the source to a local data folder if you want to run the notebooks locally with the CSVs on your own disk. It is referred to as `USE_LOCAL` in the Python and MongoDB notebooks. It is called `use_local` in the R notebook. Set it to `True` or `TRUE` and put the CSVs in a folder called `data` next to the notebook.


## Notebook 1 - SQL in R and R analytics

This notebook runs in R. By default Colab uses Python, so the runtime has to be switched first.

How to run it on Colab:

1. Open the notebook
2. Runtime, then Change runtime type, then pick R as the language, then Save
3. Runtime, then Run all
4. The first code cell installs sqldf, dplyr, ggplot2, tidyr, lubridate, and scales. This takes about two minutes on a fresh session.

What is in it:

- Loading the nine CSVs from GitHub
- Cleaning - zone fix, date parsing, integrity flag construction
- Twelve SQL queries via sqldf covering hub performance, missing deliveries, integrity issues, complaints joined back to delivery status, vehicle maintenance versus failure, worst drivers, complaint patterns, hub revenue and cost, incident backlogs, customer risk via a subquery, hourly failure rate, and app event quality
- Four statistical tests - chi-squared for complaint severity versus delivery status, one-way ANOVA for duration across hubs, Pearson correlation between driver training score and failure rate, and a logistic regression predicting delivery failure
- Eight ggplot charts with business-focused interpretations underneath each one


## Notebook 2 - Python data processing

This one runs in Python, which is Colab's default runtime, so no setup is needed.

How to run it on Colab:

1. Open the notebook
2. Runtime, then Run all

What is in it:

- Loading the nine CSVs from GitHub
- Data quality audit - referential integrity across all join paths, missing value tabulation, categorical inconsistency check, timestamp sanity check
- Cleaning - zone fix, date parsing, justified missing-value handling (impute where it makes sense, keep NaN where the gap is the finding)
- Feature engineering - delivery duration, status integrity flags (negative duration, OnTime with no completion timestamp, Failed with a customer rating), SLA breach flag, cost per km, time features, incident and complaint linkage flags
- Nine-section analysis going through hubs, services, vehicles, overrides, drivers, finance, complaints, zones, and incidents
- Nine matplotlib and seaborn charts
- Cleaned CSVs saved at the end


## Notebook 3 - MongoDB Atlas and query optimisation

This notebook runs in Python and needs a MongoDB Atlas cluster to connect to. Setup on Atlas takes about five minutes the first time. The notebook never stores the connection string in the file - it asks for it at runtime using `getpass`, so the credentials are never committed to GitHub.

### Setting up MongoDB Atlas

1. Sign up at https://cloud.mongodb.com
2. Create a project, then a free M0 cluster (the default settings are fine)
3. Go to Network Access in the left sidebar, click Add IP Address, then Allow Access from Anywhere. This adds `0.0.0.0/0` to the allow list. Colab uses different IP addresses every session so this is necessary
4. Go to Database Access, create a database user with read and write permission to any database. Save the password somewhere safe because Atlas does not show it again
5. Go back to your cluster, click Connect, choose Drivers, pick Python as the driver, and copy the connection string. It will look like this:

   ```
   mongodb+srv://your_user:<password>@cluster0.xxxxx.mongodb.net/?retryWrites=true&w=majority&appName=Cluster0
   ```

6. Replace `<password>` with the actual password from step 4. No angle brackets. The result is the full connection string you paste into the notebook

### How to run it on Colab

1. Open the notebook
2. Runtime, then Run all
3. When the connection cell runs, a text box appears. Paste your full Atlas connection string into it and press Enter. The box does not show what you type - that is normal, it is hiding the input

### What is in it

- Loading the nine CSVs from GitHub and running the same cleaning as the other notebooks
- A short section explaining why MongoDB suits this case study and why three collections were chosen
- Building three collections:
  - `service_cases` - one document per customer, with orders nested, and each order has its delivery, complaints, and incidents embedded
  - `asset_lifecycle` - one document per vehicle with performance summary and incident history embedded
  - `platform_events` - one document per session with events nested as an ordered array
- CRUD operations - insert one customer, find with filters and projections, update one customer, update many vehicles in one call, delete one customer
- Four aggregation pipelines using `$unwind`, `$group`, `$filter`, `$addFields`, and `$project`
- Indexes - single field, unique, and compound, with justification for each
- Three query optimisation tests using `explain("executionStats")` showing the COLLSCAN to IXSCAN transition for a single-field filter, a range query, and a compound query


## Main findings

The numbers below come from the actual runs in the notebooks. They line up across all three notebooks because the cleaning logic is the same.

- About 22 percent of deliveries (207 out of 950) carry at least one data integrity issue. Some are completed before they were dispatched, some are marked OnTime with no completion timestamp, and some are marked Failed but have a customer rating. These are not data entry errors - they are the visible signs of the disconnected systems the case study describes
- 24 percent of orders (300 out of 1,250) have no delivery record at all. They were booked but never reached dispatch
- Vehicles flagged InRepair are still being dispatched. They appear in 254 deliveries and fail at 30 percent compared to 8 percent for Active vehicles. The logistic regression in the R notebook confirms maintenance status is the only statistically significant predictor of failure (p less than 0.001)
- SLA breach rate is 63 to 71 percent across every service type. This is a network-wide problem, not specific to one service
- The two hubs in the Central zone (Midtown Relay and Central Core) fail at around 20 percent. The best-performing hubs are at around 9 percent
- Margins look healthy at 85 percent or above when only fuel cost is counted, but they exclude failure cost, compensation, and asset downtime. The finance director was right that the real cost of failure is hidden
- Complaint severity and delivery status are only weakly associated (chi-squared p = 0.097). If the complaint system and the operations system were properly linked, this association would be strong


## Tools used

- Python 3.12 with pandas, numpy, matplotlib, seaborn
- R 4.3 with sqldf, dplyr, ggplot2, tidyr, lubridate, scales
- MongoDB Atlas free tier (M0)
- pymongo 4.x
- Google Colab


## Submission

Submitted as part of the Databases and Analytics module, University of West London, semester 2, academic year 2025/26.
