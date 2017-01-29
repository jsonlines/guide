# JSON Lines Guide

Tutorial on streaming JSON data analysis on the command line.

## Introduction

JSON Lines (also known as Newline Delimited JSON) is a really simple way to store JSON that makes it very friendly for data processing and analysis.

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

To install the tools do the following:

1. [Install a recent Node.js](https://nodejs.org/en/download/) version (LTS Recommended) on your computer
2. Using `npm`, install the JSON Lines CLI Tools

```
$ npm install -g jsonfilter jsonmap jsonreduce jsonstats
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

## jsonmap
## jsonreduce
## jsonstats
