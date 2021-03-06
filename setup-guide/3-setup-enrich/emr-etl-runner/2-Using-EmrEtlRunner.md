<a name="top" />

[HOME](Home) » [SNOWPLOW SETUP GUIDE](Setting-up-Snowplow) » [Step 3: Setting up Enrich](Setting-up-enrich) » [Step 3.1: setting up EmrEtlRunner](Setting-up-EmrEtlRunner) [Step 3.1.1: Installing EmrEtlRunner](1-Installing-EmrEtlRunner) » Step 3.1.2: Using EmrEtlRunner

1. [Overview](#usage-overview)
2. [Commands](#commands)
3. [Checking the results](#checking)
4. [Next-steps](#next-steps)

<a name="usage-overview"/>

## 1. Overview

EmrEtlRunner works in **Rolling mode** where it processes whatever raw Snowplow event logs it finds
in the In Bucket

<a name="commands"/>

## 2. Commands

### 2.1 Run command

The most useful command is the `run` command which allows you to actually run your EMR job:

    $ ./snowplow-emr-etl-runner run

The available options are as follows:

    Usage: run [options]
        -c, --config CONFIG              configuration file
        -n, --enrichments ENRICHMENTS    enrichments directory
        -r, --resolver RESOLVER          Iglu resolver file
        -t, --targets TARGETS            targets directory
        -d, --debug                      enable EMR Job Flow debugging
        -f {enrich,shred,elasticsearch,archive_raw,rdb_load,analyze,archive_enriched},
            --resume-from                resume from the specified step
        -x {staging,enrich,shred,elasticsearch,archive_raw,rdb_load,analyze,archive_enriched},
            --skip                       skip the specified step(s)
        -i, --include {vacuum}           include additional step(s)
        -l, --lock PATH                  where to store the lock
            --consul ADDRESS             address to the Consul server

Note that the `config` and `resolver` options are mandatory.

### 2.2 Lint commands

Other useful commands include the `lint` commands which allows you to check the validity of your
resolver or enrichments with respect to their respective schemas.

If you want to lint your resolver:

    $ ./snowplow-emr-etl-runner lint resolver

The mandatory options are:

    Usage: lint resolver [options]
        -r, --resolver RESOLVER          Iglu resolver file

If you want to lint your enrichments:

    $ ./snowplow-emr-etl-runner lint enrichments

The mandatory options are:

    Usage: lint enrichments [options]
        -r, --resolver RESOLVER          Iglu resolver file
        -n, --enrichments ENRICHMENTS    enrichments directory


<a name="running"/>

<a name="checking"/>

## 3. Checking the results

Once you have run the EmrEtlRunner you should be able to manually inspect in S3 the folder specified in the `:out:` parameter in your `config.yml` file and see new files generated, which will contain the cleaned data either for uploading into a storage target (e.g. Redshift) or for analysing directly using Hive or Spark or some other querying tool on EMR.

Note: most Snowplow users run the 'spark' version of the ETL process, in which case the data generated is saved into subfolders with names of the form `part-000...`. If, however,  you are running the legacy 'hive' ETL (because e.g. you want to use Hive as your storage target, rather than Redshift, which is the only storage target the 'spark' etl currently supports), the subfolders names will be of the format `dt=...`.

## 4. Next steps

Comfortable using EmrEtlRunner? Then [schedule it][schedule] so that it regularly takes new data generated by the collector, processes it, cleans it, enriches it, and writes it back to S3.

[schedule]: 3-Scheduling-EmrEtlRunner
