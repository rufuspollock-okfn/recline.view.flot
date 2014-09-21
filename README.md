A lightweight JS library providing additional utilities and HTML-based wizard
for quickly creating [Flot charts][Flot]. Specifically:

- Utility functions to quickly generate standard charts like line and bar charts
- A HTML + javascript user interface for creating chart configurations (think:
  chart wizard in a spreadsheet app like Excel or Google Docs)

This is part of the [ReclineJS][] suite of libraries and among its other
functions it provides a Recline compatible "view" to display Flot-driven
charts.

However, the library is completely standalone and has no dependency on
[ReclineJS][].

[ReclineJS]: http://okfnlabs.org/recline/
[Flot]: http://flotcharts.com/

## Install

-Make sure you have the dependencies (note a copy of each relevant dependency
-can be found in the `vendor/` subdirectory of this repo):

* [Flot][]
* jQuery
* Backbone
* [Optional] Moment - only needed if you want to use time series data

To use the library just include the `view.flot.js` file. Copy it to your app and then do:

```
<script src="path/to/view.flot.js"></script>
```

----

# Plan for the Refactor to FlotPlus (from simple Recline Flot View)

- Invert dependency - viz stuff does not care about Recline (or even Backbone)
- Instead we have bridge code to convert from Recline data (or other sources) to our specific backend
- But what about data updating? i.e. we want to update when underlying data updates
  - IMO this is actully less useful that you think. It is true that spreadsheets do this but even there you often explicitly update in some way.
  - I also feel this is more about a UI for querying / creating series related to the underlying data (separate from graphing) - though obviously user may see this together (think of graph wizard in spreadsheet)
  - however we can still support this with some standard event stuff ... (e.g. we bind on some notification and then re-run our data generation ...)

## Details

- Have a data source object (Recline Dataset, other stuff)
- Pipe that into a Recline chart
  - Optionally pass a listener in to hear changes on data and re-run query

Key question:

- What is data structure that flot takes?
  - IIRC answer is series
  - TODO: compare with VEGA
  - See ongoing research below

=> We need to convert to that data structure

## Sketch Code Spec

```
makeFlotData: dataset => series
makeFlotOptions: 
```

Generating a plot would then look like:

```
var data = makeFlotData(dataset);
var options = makeFlotOptions(dataset (?), typeOfGraph);
var plot = $.plot(placeholder, data, options)
```

> Question / Issue: do these different steps interact? E.g. type of graph influences data generation (e.g. for bar charts we need 3rd point), plus data stuff includes things like which yaxis to use comes from some higher level spec of what we are doing.

That said we want something really simple, so we could do the simplest thing possible and then see where this does not work.

> Question: should we think about something more generic? e.g. starting with Vega and then rendering flot from that?

> Question: do we need something higher-level? Whether for hand-crafted JSON (in data package) or for a chart wizard you do want to work with common chart cone

# Research

## Vincent

"Python to Vega translator" but more accurately an API wrapper for Vega generation.

<https://github.com/wrobstory/vincent>

What's especially interesting here is that Vincent does provide a higher-level API with standard Chart types:

<http://vincent.readthedocs.org/en/latest/charts_library.html>

## Flot

https://github.com/flot/flot/blob/master/API.md#data-format

* Data is an array of series

      [ series1, series2, ... ]

* A series can either be raw data or an object with properties. The raw
  data format is an array of points:

      [ [x1, y1], [x2, y2], ... ]

  Lines and points take two coordinates. For filled lines and bars, you can specify a third coordinate which is the bottom of the filled area/bar (defaults to 0).

  Object structure is like:

      {
          color: color or number
          data: rawdata
          label: string
          lines: specific lines options
          bars: specific bars options
          points: specific points options
          xaxis: number
          yaxis: number
          clickable: boolean
          hoverable: boolean
          shadowSize: number
          highlightColor: color or number
      }

* Can specific generic stuff for series type

## Vega

https://github.com/trifacta/vega

Vega is heavily based on Grammar of Graphics + D3. As such, it does not really have a concept of chart types or anything like that. Rather it has primitives like:

* data
* data transforms
* marks
* axes
* scales
* ...

Marks are the key item here:

> Marks are the basic visual building block of a visualization. Similar to other mark-based frameworks such as Protovis, marks provide basic shapes whose properties can be set according to backing data. Mark properties can be simple constants or data fields, and Scales can be used to map from data to property values. The basic supported mark types are rectangles (rect), plotting symbols (symbol), general paths or polygons (path), circular arcs (arc), filled areas (area), lines (line), images (image) and text labels (text).

