# Week 1 Homework

## Question 1. Google Cloud SDK

Install Google Cloud SDK. What's the version you have?

To get the version, run `gcloud --version`

```txt
Google Cloud SDK 371.0.0
bq 2.0.73
core 2022.01.28
gsutil 5.6
```

## Google Cloud account

Create an account in Google Cloud and create a project.

## Question 2. Terraform

Now install terraform and go to the terraform directory (`week_1_basics_n_setup/1_terraform_gcp/terraform`)

After that, run

- `terraform init`
- `terraform plan`
- `terraform apply`

Apply the plan and copy the output (after running `apply`) to the form.

It should be the entire output - from the moment you typed `terraform init` to the very end.

```txt
❯ terraform init

Initializing the backend...

Initializing provider plugins...
- Reusing previous version of hashicorp/google from the dependency lock file
- Using previously-installed hashicorp/google v4.9.0

Terraform has been successfully initialized!

You may now begin working with Terraform. Try running "terraform plan" to see
any changes that are required for your infrastructure. All Terraform commands
should now work.

If you ever set or change modules or backend configuration for Terraform,
rerun this command to reinitialize your working directory. If you forget, other
commands will detect it and remind you to do so if necessary.

❯ terraform plan
var.project
  Your GCP Project ID

  Enter a value: apt-reason-338813

google_storage_bucket.data-lake-bucket: Refreshing state... [id=dtc_data_lake_apt-reason-338813]
google_bigquery_dataset.dataset: Refreshing state... [id=projects/apt-reason-338813/datasets/trips_data_all]

Note: Objects have changed outside of Terraform

Terraform detected the following changes made outside of Terraform since the
last "terraform apply":

  # google_bigquery_dataset.dataset has changed
  ~ resource "google_bigquery_dataset" "dataset" {
        id                              = "projects/apt-reason-338813/datasets/trips_data_all"
      + labels                          = {}
        # (10 unchanged attributes hidden)

        # (4 unchanged blocks hidden)
    }

  # google_storage_bucket.data-lake-bucket has changed
  ~ resource "google_storage_bucket" "data-lake-bucket" {
        id                          = "dtc_data_lake_apt-reason-338813"
      + labels                      = {}
        name                        = "dtc_data_lake_apt-reason-338813"
        # (9 unchanged attributes hidden)


        # (2 unchanged blocks hidden)
    }


Unless you have made equivalent changes to your configuration, or ignored the
relevant attributes using ignore_changes, the following plan may include actions
to undo or respond to these changes.

────────────────────────────────────────────────────────────────────────────────

No changes. Your infrastructure matches the configuration.

Your configuration already matches the changes detected above. If you'd like to
update the Terraform state to match, create and apply a refresh-only plan:
  terraform apply -refresh-only


❯ terraform apply
var.project
  Your GCP Project ID

  Enter a value: apt-reason-338813

google_storage_bucket.data-lake-bucket: Refreshing state... [id=dtc_data_lake_apt-reason-338813]
google_bigquery_dataset.dataset: Refreshing state... [id=projects/apt-reason-338813/datasets/trips_data_all]

Note: Objects have changed outside of Terraform

Terraform detected the following changes made outside of Terraform since the last "terraform apply":

  # google_bigquery_dataset.dataset has changed
  ~ resource "google_bigquery_dataset" "dataset" {
        id                              = "projects/apt-reason-338813/datasets/trips_data_all"
      + labels                          = {}
        # (10 unchanged attributes hidden)

        # (4 unchanged blocks hidden)
    }

  # google_storage_bucket.data-lake-bucket has changed
  ~ resource "google_storage_bucket" "data-lake-bucket" {
        id                          = "dtc_data_lake_apt-reason-338813"
      + labels                      = {}
        name                        = "dtc_data_lake_apt-reason-338813"
        # (9 unchanged attributes hidden)


        # (2 unchanged blocks hidden)
    }


Unless you have made equivalent changes to your configuration, or ignored the relevant attributes using ignore_changes, the following plan may
include actions to undo or respond to these changes.

──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────

No changes. Your infrastructure matches the configuration.

Your configuration already matches the changes detected above. If you'd like to update the Terraform state to match, create and apply a
refresh-only plan:
  terraform apply -refresh-only

Apply complete! Resources: 0 added, 0 changed, 0 destroyed.
```

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

```sql

-- my solution
SELECT
   COUNT(*)
FROM
   yellow_taxi_trips
WHERE
   EXTRACT(MONTH FROM tpep_pickup_datetime)::INTEGER = 1
  AND EXTRACT(DAY FROM tpep_pickup_datetime)::INTEGER = 15

-- zoomcamp
SELECT
   COUNT(*)
FROM
   yellow_taxi_trips
WHERE
    tpep_pickup_datetime::date = '2021-01-15'
```

```txt
+-------+
| count |
|-------|
| 53024 |
+-------+
```

## Question 4. Largest tip for each day

Find the largest tip for each day.
On which day it was the largest tip in January?

Use the pick up time for your calculations.

(note: it's not a typo, it's "tip", not "trip")

```sql

-- my solution
SELECT
   EXTRACT(DAY FROM tpep_pickup_datetime)::INTEGER as pickup_day,
   MAX(tip_amount) as tip
FROM
   yellow_taxi_trips
GROUP BY
   pickup_day
ORDER BY tip desc
LIMIT 1;

-- zoomcamp solution
SELECT
   date_trunc('day', tpep_pickup_datetime) as pickup_day,
   MAX(tip_amount) as max_tip
FROM
   yellow_taxi_trips
GROUP BY
   pickup_day
ORDER BY max_tip DESC
LIMIT 1;
```

```txt
+---------------------+---------+
| pickup_day          | max_tip |
|---------------------+---------|
| 2021-01-20 00:00:00 | 1140.44 |
+---------------------+---------+
```

## Question 5. Most popular destination

What was the most popular destination for passengers picked up
in central park on January 14?

Use the pick up time for your calculations.

Enter the zone name (not id). If the zone name is unknown (missing), write "Unknown"

```sql
SELECT
   COALESCE(dozones."Zone", 'Unknown') as zone,
   COUNT(*) as count_trips
FROM
   yellow_taxi_trips trip
   INNER JOIN zones as puzones
   ON trip."PULocationID" = puzones."LocationID"
   LEFT JOIN zones as dozones
   ON trip."DOLocationID" = dozones."LocationID"
WHERE
   puzones."Zone" ilike '%central park%'
   and tpep_pickup_datetime::date = '2021-01-14'
GROUP BY
   1
ORDER BY
   count_trips DESC
LIMIT 5
```

```txt
+-----------------------+-------------+
| zone                  | count_trips |
|-----------------------+-------------|
| Upper East Side South | 97          |
| Upper East Side North | 94          |
| Lincoln Square East   | 83          |
| Upper West Side North | 68          |
| Upper West Side South | 60          |
+-----------------------+-------------+
```

## Question 6. Most expensive locations

What's the pickup-dropoff pair with the largest
average price for a ride (calculated based on `total_amount`)?

Enter two zone names separated by a slash

For example:

"Jamaica Bay / Clinton East"

If any of the zone names are unknown (missing), write "Unknown". For example, "Unknown / Clinton East".

```sql
SELECT
   CONCAT(coalesce(pickup."Zone", 'Unknown'),
        ' / ',
        coalesce(dropoff."Zone", 'Unknown')),
   AVG(trips.total_amount) as avg_price_ride
FROM
   yellow_taxi_trips trips
   LEFT JOIN
      zones pickup
      ON pickup."LocationID" = trips."PULocationID"
   LEFT JOIN
      zones dropoff
      ON dropoff."LocationID" = trips."DOLocationID"
GROUP BY
   1
ORDER BY
   avg_price_ride DESC
LIMIT 5
```

```txt
+-----------------------------------------------+----------------+
| concat                                        | avg_price_ride |
|-----------------------------------------------+----------------|
| Alphabet City / Unknown                       | 2292.4         |
| Union Sq / Canarsie                           | 262.852        |
| Ocean Hill / Unknown                          | 234.51         |
| Long Island City/Hunters Point / Clinton East | 207.61         |
| Boerum Hill / Woodside                        | 200.3          |
+-----------------------------------------------+----------------+
```

## Submitting the solutions

- Form for submitting: <https://forms.gle/yGQrkgRdVbiFs8Vd7>
- You can submit your homework multiple times. In this case, only the last submission will be used.

Deadline: 26 January (Wednesday), 22:00 CET

## Solution

Here is the solution to questions 3-6: [video](https://www.youtube.com/watch?v=HxHqH2ARfxM&list=PL3MmuxUbc_hJed7dXYoJw8DoCuVHhGEQb)
