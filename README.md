# gcs-inventory-loader

Load your GCS bucket inventory into BigQuery fast with this tool.

## Introduction

It can be very useful to have an inventory of your GCS objects and their metadata, particularly in a powerful database like BigQuery. The GCS listing API supports filtering by prefixes, but more complex queries can't be done via API. Using a database, you can find out lots of information about the data you have in GCS, such as finding very large objects, very old or stale objects, etc.

This utility will help you bulk load an object listing to stdout, or directly into BigQuery. It doesn't help with keeping your database incrementally up-to-date, though that is easily achieved with either [Audit Logging](https://cloud.google.com/storage/docs/audit-logs) or [PubSub Notifications](https://cloud.google.com/storage/docs/pubsub-notifications).

The implementation here takes the approach of listing buckets and sending each page to a worker in a thread pool for processing and streaming into BigQuery. Throughput rates of 15s per 100,000 objects have been achieved with moderately sized (32 vCPU) virtual machines. This works out to 2 minutes and 30 seconds per million objects. Note that this throughput is _per process_ -- simply shard the bucket namespace across multiple projects to increase this throughput.

## Installation

  1) Download this repository and `cd` into it.
  2) Run `pip install .`. Optionally, add the `-e` switch. This will allow you to make modifications if you'd like.

## Usage

First, configure the config file. You will find a `./default.cfg` file in the root of this repo. Each field is documented. Be sure to fill in the `PROJECT` value, and if streaming to BigQuery directly, the `DATASET_NAME` and `INVENTORY_TABLE` values.

Now, you are ready to run the command. You can get general help by just running `gcs_inventory`.

``` shell
$ gcs_inventory
Usage: gcs_inventory [OPTIONS] COMMAND [ARGS]...
```

### Streaming to BigQuery

If you've configured the config file correctly, you should be able to get your bucket inventory loaded with a simple command:

``` shell
gcs_inventory load
```

Note that by default, this will load an inventory of all objects for _all buckets_ in your project. To restrict this further, you can use the bucket list argument and/or the filter option:

``` shell
gcs_inventory load bucket1 bucket2
```

``` shell
gcs_inventory load bucket1 -p stuffICareAbout/
```

### Writing to stdout

You can also output the records in newline-delimited JSON for use with another database, or for bulk loading into BigQuery via GCS. For example:

``` shell
gcs_inventory cat > inventory.ldjson
```

You can combine this command with gsutil's ability to read from stdin for a stream directly into GCS:

``` shell
gcs_inventory cat | gsutil cp - gs://mybucket/inventory.ldjson
```

## Important Disclaimer

This code is written by a Googler, but this project is not supported by Google in any way. As the `LICENSE` file says, this work is offered to you on an "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied, including, without limitation, any warranties or conditions of TITLE, NON-INFRINGEMENT, MERCHANTABILITY, or FITNESS FOR A PARTICULAR PURPOSE. You are solely responsible for determining the appropriateness of using or redistributing the Work and assume any risks associated with Your exercise of permissions under this License.

## License

Apache 2.0 - See [the LICENSE](/LICENSE) for more information.

## Copyright

Copyright 2020 Google LLC.
