# JSON Lines Guide

Tutorial on streaming JSON data analysis on the command line.

#### Table of Contents

- [Introduction](#introduction)
- [Installation](#installing-json-lines-tools)
- [`jsonfilter`](#jsonfilter)
- [`jsonmap`](#jsonmap)
- [`jsonstats`](#jsonstats)
- [`ndjson-reduce`](#ndjson-reduce)
- [useful pipelines](#useful_pipelines) (WIP)

## Introduction

[JSON Lines](http://jsonlines.org/) (also known as [Newline Delimited JSON](http://ndjson.org/)) is a really simple way to store JSON that makes it very friendly for data processing and analysis. To store data in JSON Lines format you simply write one JSON stringified object per line.

Usually you would store multiple JSON objects in a single, pretty printed JSON structure in a file like this:

```js
[
  {
    "url": "https://tigerweb.geo.census.gov/arcgis/services/TIGERweb/tigerWMS_Census2010/MapServer/WmsServer",
    "date": "2017-01-27T03:41:03.083Z"
  },
  {
    "url": "https://tigerweb.geo.census.gov/arcgis/rest/services/TIGERweb/PUMA_TAD_TAZ_UGA_ZCTA/MapServer",
    "date": "2017-01-27T03:41:03.121Z"
  }
]
```

The JSON Lines version of the same information above would be this:

```
{"url":"https://tigerweb.geo.census.gov/arcgis/services/TIGERweb/tigerWMS_Census2010/MapServer/WmsServer","date":"2017-01-27T03:41:03.083Z"}
{"url":"https://tigerweb.geo.census.gov/arcgis/rest/services/TIGERweb/PUMA_TAD_TAZ_UGA_ZCTA/MapServer","date":"2017-01-27T03:41:03.121Z"}
```

This is a little harder to read but the advantage is that it's much easier to process on the command line using streams, especially if instead of two objects you had a million!

The main advantage of JSON Lines is that you can process each row of the file one at a time without having to read the entire file into memory, which is very important for larger datasets.

This guide assumes you are on a UNIX machine, but all of the `json*` tools work on Windows as well.

## Installing JSON Lines Tools

The `jsonlines` GitHub organization houses a collection of handy modules for processing JSON Lines data on the command line.

You should also check out the `ndjson-cli` suite from Mike Bostock which offers even more tools, available at https://github.com/mbostock/ndjson-cli, the concepts in this guide are the same.

To install the tools do the following:

1. [Install a recent Node.js](https://nodejs.org/en/download/) version (LTS Recommended) on your computer
2. Using `npm`, install the JSON Lines CLI Tools

```
$ npm install -g jsonfilter jsonmap jsonstats
```

If you receive permissions errors, you may need to use `sudo` or [change your permissions](https://docs.npmjs.com/getting-started/fixing-npm-permissions).

## jsonfilter

The [`jsonfilter`](https://github.com/jsonlines/jsonfilter) command takes in a stream of JSON Lines and based on a filter expression exports a subset of data out as JSON Lines.

For example purposes this repository contains the metadata for 100 Data.gov datasets in JSON Lines format.

You can download it like this:

```
curl https://raw.githubusercontent.com/jsonlines/guide/master/datagov100.json > data.json
```

Then you can use `jsonfilter` to find the names of all the datasets:

```
$ cat data.json | jsonfilter name
"va-national-formulary"
"safer-company-snapshot-safer-company-snapshot"
"fatality-analysis-reporting-system-fars-ftp-raw-data"
"tiger-line-shapefile-2013-state-alabama-current-county-subdivision-state-based"
"tiger-line-shapefile-2013-state-virginia-current-county-subdivision-state-based"
"national-motor-vehicle-crash-causation-survey-nmvccs-nmvccs-xml-case-viewer"
...
```

You should see output similar to above, lots of strings. This is also valid JSON Lines output. Lines can be any JSON type: Strings, Numbers, Objects, Arrays.

You can also do filters like this:

```
$ cat data.json | jsonfilter organization.name
"va-gov"
"dot-gov"
"dot-gov"
"census-gov"
"census-gov"
"dot-gov"
"census-gov"
"census-gov"
...
```

You can use `sort` and `uniq` to find out which organization is most active.

```
$ cat data.json | jsonfilter organization.name | sort | uniq -c
  58 "census-gov"
  14 "dot-gov"
   2 "gsa-gov"
   1 "nsf-gov"
   8 "opm-gov"
   1 "ssa-gov"
  11 "usgs-gov"
   5 "va-gov"
```

You can view more examples in the [jsonfilter README](https://github.com/jsonlines/jsonfilter)

## jsonmap

This command lets you morph data from one form into another. As opposed to `jsonfilter` which takes a JSON selector expression, `jsonmap` takes a short JavaScript expression as input that gets applied to each row of incoming JSON, and returns a new Object.

For example if we wanted to grab just a couple of fields from each Data.gov metadata item we could do this:

```
$ cat data.json | jsonmap "{name: this.name, organization: this.organization.name}"
{"name":"va-national-formulary","organization":"va-gov"}
{"name":"safer-company-snapshot-safer-company-snapshot","organization":"dot-gov"}
{"name":"fatality-analysis-reporting-system-fars-ftp-raw-data","organization":"dot-gov"}
...
```

The variable `this` in the expression has the data for the row of JSON. You can also use Template Strings:

```
$ cat data.json | jsonmap '`Notes: ${this.notes.slice(0, 60)}...`'
"Notes: The VA National Formulary is a listing of products (drugs an..."
"Notes: The Company Snapshot is a concise electronic record of compa..."
"Notes: The program collects data for analysis of traffic safety cra..."
...
```

If your expression starts with something other than the above two things (objects or template literals) it will simply be used as the function body that gets executed on each row. You can either modify `this` which will get returned at the end of the function:

```
$ cat data.json | jsonmap "if (this.maintainer) this.maintainer = this.maintainer.toUpperCase()"
{"license_title":"Creative Commons CCZero","maintainer":"DON LEES","relationships_...
{"license_title":"Other License Specified","maintainer":"JAMIE VASSER","relationsh...
{"license_title":"U.S. Government Work","maintainer":"LIXIN ZHAO","relationships_a...
...
```

Or return your own custom data:

```
$ cat data.json | jsonmap "if (this.license_id === 'cc-zero') { return 'Open' } else { return 'Closed'}"
"Open"
"Closed"
"Closed"
...
```

For more examples you can check out the [jsonmap README](https://github.com/jsonlines/jsonmap/).

## jsonstats

Sometimes you want to get numerical statistics from your data. For example, the metadata includes some page view count metrics:

```
$ cat data.json | jsonfilter tracking_summary.total | head -n5
11754
2241
1925
1139
3247
```

You can pipe a stream of numbers to `jsonstats` and when you finish piping, some summary statistics will be printed out:

```
$ cat data.json | jsonfilter tracking_summary.total | jsonstats | json
{
  "max": 11754,
  "min": 3,
  "n": 100,
  "_geometric_mean": 2.0157057404897112e+252,
  "_reciprocal_sum": 1.2014730007931738,
  "mean": 741.4800000000001,
  "ss": 165161200.95999998,
  "sum": 74148,
  "_seen_this": 1,
  "_mode": 11754,
  "_mode_valid": true,
  "variance": 1651612.0095999998,
  "standard_deviation": 1285.1505785704646,
  "geometric_mean": 333.4604034827265,
  "harmonic_mean": 83.23116702080132,
  "mode": 11754,
  "_max_seen": 1,
  "_last": 189
}
```

## ndjson-reduce

If you want to take some JSON Lines output and combine the lines into a normal single JSON array of objects that you can use with `JSON.parse` in your programs, you can use the `ndjson-reduce` command.

The `ndjson-reduce` command gets installed if you run `npm install ndjson-cli -g`, along with some other great tools from the [NDJSON CLI](http://npmjs.org/ndjson-cli) package by Mike Bostock, which offers similar functionality to the other tools in this guide.

To generate a single JS array with all of the unique organization names:

```
$ cat data.json | jsonfilter organization.name | sort | uniq | ndjson-reduce
["census-gov","dot-gov","gsa-gov","nsf-gov","opm-gov","ssa-gov","usgs-gov","va-gov"]
```

## useful pipelines

Here are some more tools that work really well in combination with JSON Lines tools:

### `head`

Built in to Unix. Lets you see the beginning N lines of a stream:

```
$ cat data.json | head -n1
{"license_title":"Creative Commons CCZero","maintainer":"Don Lees","relationships_...
```

### `json`

Available as `npm install json -g` from npm, it pretty prints a JSON object!

```
$ cat data.json | head -n1 | json
{
  "license_title": "Creative Commons CCZero",
  "maintainer": "Don Lees",
  "relationships_as_object": [],
  "private": false,
...
```

### `wc`

Built in to Unix for doing word counts, but can also count lines, very useful for working with JSON Lines to count how many lines are in a file or filter

```
$ cat data.json | wc -l
100
```

### `grep`

Built in to Unix, useful if you just want to filter lines based on a regular expression

```
$ cat data.json | grep CCZero | wc -l
8
```
