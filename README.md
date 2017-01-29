# JSON Lines Guide

Tutorial on streaming JSON data analysis on the command line

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

## Installing JSON Lines Tools

The `jsonlines` GitHub organization houses a collection of handy modules for processing JSON Lines data on the command line.

To install the tools do the following:

1. [Install a recent Node.js](https://nodejs.org/en/download/) version (LTS Recommended) on your computer
2. Using `npm`, install the JSON Lines CLI Tools

```
npm install -g jsonfilter jsonmap jsonreduce jsonstats
```

If you receive permissions errors, you may need to use `sudo` or [change your permissions](https://docs.npmjs.com/getting-started/fixing-npm-permissions).
