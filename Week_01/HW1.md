## Week 1 Homework

In this homework we'll prepare the environment 
and practice with terraform and SQL

## Question 1. Google Cloud SDK

Install Google Cloud SDK. What's the version you have? 

To get the version, run `gcloud --version`

A: Google Cloud SDK 368.0.0 , bq 2.0.72, core 2022.01.07, gsutil 5.6


## Google Cloud account 

Create an account in Google Cloud and create a project.


## Question 2. Terraform 

Now install terraform and go to the terraform directory (`week_1_basics_n_setup/1_terraform_gcp/terraform`)

After that, run

* `terraform init`
* `terraform plan`
* `terraform apply` 

Apply the plan and copy the output (after running `apply`) to the form

A : No changes. Your infrastructure matches the configuration.
Terraform has compared your real infrastructure against your configuration as needed.
Apply complete! Resources: 0 added, 0 changed, 0 destroyed.


## Prepare Postgres 

Run Postgres and load data as shown in the videos

We'll use the yellow taxi trips from January 2021:

```bash
wget https://s3.amazonaws.com/nyc-tlc/trip+data/yellow_tripdata_2021-01.csv
```

You will also need the dataset with zones:

```bash 
wget https://s3.amazonaws.com/nyc-tlc/misc/taxi+_zone_lookup.csv
```

Download this data and put it to Postgres

## Question 3. Count records 

How many taxi trips were there on January 15?

Consider only trips that started on January 15.

```
SELECT COUNT(*)
FROM yellow_taxi_trips AS ytr
WHERE ytr.tpep_pickup_datetime::date = '2021-01-15';
```

A : 53042

## Question 4. Average

Find the largest tip for each day. 
On which day it was the largest tip in January?

Use the pick up time for your calculations.

(note: it's not a typo, it's "tip", not "trip")

```
SELECT ytr.tpep_pickup_datetime::date AS date, 
		SUM(tip_amount) AS total_tip
FROM yellow_taxi_trips AS ytr
GROUP BY ytr.tpep_pickup_datetime::date
ORDER BY total_tip DESC;
```

A : '2021-01-15'

## Question 5. Most popular destination

What was the most popular destination for passengers picked up 
in central park on January 14?

Use the pick up time for your calculations.

Enter the zone name (not id). If the zone name is unknown (missing), write "Unknown" 

```
SELECT ytr."DOLocationID",tz.zone_name, COUNT(*) AS total_do
FROM taxi_zone AS tz
INNER JOIN yellow_taxi_trips AS ytr
ON tz."LocationID" = ytr."DOLocationID"
WHERE ytr."PULocationID" = 43
AND ytr.tpep_pickup_datetime::date = '2021-01-14'
GROUP BY ytr."DOLocationID", tz.zone_name
ORDER BY total_do DESC;
```

A : Upper East Side South

## Question 6. 

What's the pickup-dropoff pair with the largest 
average price for a ride (calculated based on `total_amount`)?

Enter two zone names separated by a slash

For example:

"Jamaica Bay / Clinton East"

If any of the zone names are unknown (missing), write "Unknown". For example, "Unknown / Clinton East". 

```
SELECT ytr."PULocationID", 
	ytr."DOLocationID", 
	tz_pu.zone_name AS zonename_pu, 
	COALESCE(tz_do.zone_name, 'Unknown') AS zonename_do,
	AVG(total_amount) AS avg_price
FROM yellow_taxi_trips AS ytr
INNER JOIN taxi_zone AS tz_pu
ON tz_pu."LocationID" = ytr."PULocationID"
INNER JOIN taxi_zone AS tz_do
ON tz_do."LocationID" = ytr."DOLocationID"
GROUP BY ytr."PULocationID", ytr."DOLocationID",tz_pu.zone_name, tz_do.zone_name
ORDER BY avg_price DESC
```

A : "Alphabet City / Unknown"