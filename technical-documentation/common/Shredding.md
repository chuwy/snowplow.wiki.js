[**HOME**](Home) » [**SNOWPLOW TECHNICAL DOCUMENTATION**](Snowplow-technical-documentation) » [Enrichment](Enrichment) » Shredding

## Overview

Snowplow has a Shredding process which consists of three phases:

1. Extracting unstructured event JSONs and context JSONs from enriched event files into their own files
2. Removing endogenous duplicate records, which are sometimes introduced within the Snowplow pipeline (feature added to r76)
3. Loading those files into corresponding tables in Redshift

Postgres does not support copying data in JSON format, so you cannot load Snowplow JSONs into Postgres. However, if you are using Postgres, you must still run the shred step because it also creates the TSV which is loaded into the `atomic.events` table.

## Technical architecture

The shredding flow and main components are highlighted in blue on the right-hand side of this technical architecture:

[[/technical-documentation/images/shredding-architecture.png]]

## Main components

### 0. Iglu

Iglu is a schema repository technology which holds the JSON Schemas against which self-describing events (also called unstructured events) and context JSONs are validated.

For more information on Iglu, see the [Iglu wiki][iglu-wiki].

### 1. Relational Database Shredder

Relational Database Shredder (RDB Shredder for short) is a dedicated Spark job performing JSON
validation and extraction. This is a five step process:

1. Reads Snowplow enriched events from S3
2. Extracts any unstructured event JSONs and context JSONs found
3. Validates that these JSONs conform to schema
4. Adds metadata to these JSONs to track their origins
5. Writes these JSONs out to nested folders dependent on their schema

Configuring this is covered in [Configuring shredding](6-Configuring-shredding).

As of release R76, removing endogenous duplicate records was also introduced as a phase in Shredding
process. It does not require an additional configuration. RDB Shredder de-duplicates “natural
duplicates” - i.e. events which share the same event ID and the same event fingerprint, meaning that
they are semantically identical to each other.

For a given ETL run of events being processed, RDB Shredder will now keep only one out of each group
of natural duplicates; all others will be discarded.

More technical information about RDB Shredder can be found on the dedicated
[RDB Shredder](Relational-Database-Shredder) page.

### 2. Relational Database Loader

The RDB Loader provides a way to load shredded types into the corresponding tables in Redshift,
using the Redshift's native [`COPY FROM JSON`](http://docs.aws.amazon.com/redshift/latest/dg/copy-usage_notes-copy-from-json.html)
functionality. This is a multi-step process:

* Find folders of shredded types in S3
* For each folder of shredded types:
  * Find the JSON Paths file that corresponds to the shredded type
  * Determine the Redshift tablename from the shredded type
  * Load the shredded type files into the Redshift table using the JSON Paths file

Configuring this is covered in [Loading shredded types](4-Loading-shredded-types).

[iglu-wiki]: https://github.com/snowplow/iglu/wiki
