# lakehouse-quality-gate

**An active data-quality gate for a dbt + Iceberg lakehouse: it doesn't just document expectations, it blocks the pipeline and pages someone when they're violated.**

![lakehouse-quality-gate](assets/hero.png)

## The problem

Most "data quality" in a lakehouse is passive. Teams write dbt "data_tests" or Great Expectations suites, run them, and get a report -- after the bad data has already landed in the marts your dashboards and ML features read from. A null-flooded revenue column or a retried extraction job that silently duplicated half a day's records doesn't throw an exception; it just quietly corrupts everything downstream until someone notices the numbers look wrong, days later.

This project treats data quality as an enforcement point, not a report. A Great Expectations checkpoint sits physically between the dbt staging/intermediate layers and the marts. If it fails, the pipeline stops -- marts are never built on top of bad data -- and a structured alert payload, naming the exact failing expectations and a sample of the offending row IDs, goes out over a webhook. It's the difference between "our tests are red" (nobody's watching) and "the pipeline is down" (somebody gets paged).

## How it works

```mermaid
flowchart TD
    A[fixtures/*.csv<br/>synthetic public trip data] --> B[ingestion/land_raw_data.py<br/>lands real Apache Iceberg tables]
    B -->