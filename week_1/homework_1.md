# Homework 1

## Requirements
1. Installed Docker
2. Installed psql

## Preparations
1. Download required files (gzipped CSVs)
2. Unzip gzipped CSVs: `gzip -d <filename>`

## Run postresql
1. Run container

```bash
docker-compose up -d
```

## Create tables

1. Create a table for trips
```sql
CREATE TABLE "green_tripdata"(
  "vendor_id" bigint,
  "lpep_pickup_datetime" TIMESTAMP,
  "lpep_dropoff_datetime" TIMESTAMP,
  "store_and_fwd_flag" TEXT,
  "ratecode_id" bigint,
  "pu_location_id" bigint,
  "do_location_id" bigint,
  "passenger_count" INTEGER,
  "trip_distance" float8,
  "fare_amount" float8,
  "extra" float8,
  "mta_tax" float8,
  "tip_amount" float8,
  "tolls_amount" float8,
  "ehail_fee" float8,
  "improvement_surcharge" float8,
  "total_amount" float8,
  "payment_type" float8,
  "trip_type" bigint,
  "congestion_surcharge" TEXT
);

```

2. Create a table for zones

```sql
CREATE TABLE "taxi_zone_lookup"(
    "location_id" bigint,
    "borough" text,
    "zone" text,
    "service_zone" text
);
```

## Import files

```bash
psql -h 0.0.0.0 -d dev --user=postgres -c "\copy green_tripdata FROM './green_tripdata_2019-01.csv' delimiter ',' CSV HEADER;"
psql -h 0.0.0.0 -d dev --user=postgres -c "\copy taxi_zone_lookup FROM './taxi_zone_lookup.csv' delimiter ',' CSV HEADER;"
```

## Questions

### Question 1. Knowing docker tags
**Answer:** --iidfile string

### Question 2. Understanding docker first run
**Answer:** 3

### Question 3: Count records
How many taxi trips were totally made on January 15? \
Tip: started and finished on 2019-01-15.

```sql
select count(*) from green_tripdata
where 1 = 1
and date_trunc('day', lpep_pickup_datetime) = '2019-01-15' 
and  date_trunc('day', lpep_dropoff_datetime) = ' 2019-01-15'
```

**Answer:** 20530

### Question 4: Largest trip for each day
Which was the day with the largest trip distance. \
 Use the pick up time for your calculations.

```sql
select date_trunc('day', lpep_pickup_datetime) as day, max(trip_distance) from green_tripdata
group by 1
order by 2 desc
limit 1
```
**Answer:** 2019-01-15

### Question 5: The number of passengers
In 2019-01-01 how many trips had 2 and 3 passengers?

```sql
select passenger_count, count(*) from green_tripdata
where 1 = 1
and  date_trunc('day', lpep_pickup_datetime) = '2019-01-01'
and passenger_count in (3, 2)
group by 1
order by 1;
```

**Answer:** (2, 1282), (3, 254)

### Question 6:  Largest tip
For the passengers picked up in the Astoria Zone which was the drop off zone that had the largest tip? We want the name of the zone, not the id.
```sql
select z.zone from (
  select do_location_id, max(tip_amount) from green_tripdata as t
  left join taxi_zone_lookup as z on t.pu_location_id = z.location_id
  where z.zone = 'Astoria'
  group by 1
  order by 2 desc
  limit 1
) as largest_tip left join taxi_zone_lookup as z on largest_tip.do_location_id = z.location_id
```
