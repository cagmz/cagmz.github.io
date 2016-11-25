---
layout: post
title: How to create a Time Series graph with Angular and D3
excerpt: "I recently implemented a time series graph in a web app that uses Angular on the front end and I wanted to share what I learned."
modified: 2016-11-24
tags: [tutorial, Angular, D3, JavaScript]
comments: true
---

While building my senior capstone project, [iSprinkle](https://github.com/cagmz/iSprinkle), I needed a way to display a time series graph of data. 
A working demo can be found on my [Codepen](https://codepen.io/cagmz/pen/qqRGoY) and a gist over on [Github](https://gist.github.com/cagmz/eca1fae44b0c4d9de522104607e4f474). Here's what I learned.

Implementing a time series graph into an existing Angular app requires a few dependecies:

* [D3](https://github.com/cagmz/iSprinkle), a data visualization library
* [NVD3](http://nvd3.org/), a reusable chart library for D3
* [Angular-nvD3](http://krispo.github.io/angular-nvd3/#/), for AngularJS directives
* [Moment.js](http://momentjs.com/), a library for working with JavaScript dates/times

Note: As of this writing, nvd3 supports D3 versions 3.5.3 and above, but not D3 4.x .

In this minimal example, our Angular app and controllers are named like so:

~~~ html
<body>
	<div ng-app="nvd3_timeseries">
		<div ng-controller="demo">
		<nvd3 options="options" data="data"></nvd3>
		</div>
	</div>
</body>
~~~

However, because some dependecies depend on others, we'll need to make that they load in a particular order:

~~~ html
<head>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/moment.js/2.16.0/moment.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/d3/3.5.17/d3.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/angular.js/1.5.7/angular.js"></script>
    <link rel="stylesheet" type="text/css" href="https://cdnjs.cloudflare.com/ajax/libs/nvd3/1.8.4/nv.d3.min.css">
    <script src="https://cdnjs.cloudflare.com/ajax/libs/nvd3/1.8.4/nv.d3.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/angular-nvd3/1.0.9/angular-nvd3.min.js"></script>
</head>
~~~

Finally, most of the code for the graph is found within the `home` controller. In `$scope.options`, we'll find the configuration options for the chart. The actual data for the chart is stored in `$scope.data`, which should contain one or more time series objects in this format:

~~~ javascript
// xy_coordinate_pairs is a time series and should be an array of x, y coordinate pairs
// {x: xVal, y: yVal }
return [{
  values: xy_coordinate_pairs,
  key: 'Chart Key',
  color: '#1b75ba'
}];
~~~

In this example, the `generateTimeSeries` function is called to 

~~~ javascript
// 1) Declare app
var nvd3_timeseries = angular.module('nvd3_timeseries', ['nvd3']);

// 2) Declare controller
nvd3_timeseries.controller('demo', ['$scope', function($scope) {

    // 3) Chart configuration
    $scope.options = {
        chart: {
            type: 'lineChart',
            height: 450,
            margin: {
                top: 20,
                right: 20,
                bottom: 40,
                left: 55
            },
            x: function(d) {
            /*  
            Moment.js will attempt to parse the x coordinate 
            and return a value in milliseconds
            */
                return moment.utc(d.x).valueOf();
            },
            y: function(d) {
                return d.y;
            },
            useInteractiveGuideline: true,
            xAxis: {
                axisLabel: 'Date',
                tickFormat: (function(d) {
                    // 3.1) Format the x ticks
                    return d3.time.format('%-m/%-d/%-Y')(new Date(d));
                })
            },
            yAxis: {
                axisLabel: 'Cats',
                axisLabelDistance: -10
            },
            callback: function(chart) {}
        },
        title: {
            enable: true,
            text: 'Time Series graph'
        }
    };

    $scope.dateRange = function dateRange(startDate, endDate) {
        // http://stackoverflow.com/a/23796069
        var dates = [];

        var currDate = startDate.clone().startOf('day');
        var lastDate = endDate.clone().startOf('day');

        while (currDate.add(1, 'days').diff(lastDate) <= 0) {
            dates.push(currDate.clone().toDate());
        }

        return dates;
    };

    // 4) Chart data
    $scope.data = generateTimeSeries();

    function generateTimeSeries() {
        // Generate an array of dates for plotting
        var dateRange = $scope.dateRange(moment().subtract(7, 'days'), moment());

        // Plottable will be an array of (x, y) objects
        var plottable = dateRange;

        plottable.forEach(function to_utc(date, index) {
            console.log('a[' + index + '] = ' + date);
            // set x value to date and y to a random value
            var rand = Math.round((Math.random() * (100 - 1) + 1));
            var point = {
                x: date,
                y: rand
            };
            // Replace original element
            plottable[index] = point;
        });

        // Return the plottable array for plotting
        return [{
            values: plottable,
            key: 'Cats',
            color: '#1b75ba'
        }];
    };
}]);
~~~

As an aside, although this chart is initiated with data instantly, we can set an empty data value like so:


~~~ javascript
$scope.data = [{
    "key": "Some key",
    "values": []
}];
~~~

