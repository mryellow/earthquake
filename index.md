---
title: Earthquakes in Asia Pacific since 2009
layout: mapvis
---

<script src="{{ site.baseurl }}/js/d3.v3.min.js"></script>


<script type="text/javascript">

var mapconf = {
    extent: [
      {lat: 0.00, lon: 110.00},
      {lat: -45.00, lon: 175.00}
    ],
    mapid: "mr-yellow.jn4j6iof"
  },
  visconf = {
    duration: 4 * 60 * 1000,
    radExp: 5,
    radExtent: [20, 300],
    durationEntent: [300, 1000],
    cirPoint: {
    radius:  4
  },
  txtPoint: {
    margin: {
      top: 80,
      left: 0
    },
    fontsize: 50
  },
  rectYear: {
    width:  280,
    height: 70,
    margin: 10
  },
  txtYear: {
    margin: {
      top: 50,
      left: 30
    },
    fontsize: 50
  },

  rectTip: {
    width:  300,
    height: 75,
    margin: {
      top: -75/2,
      left: 25
    }
  },
  txtTip: {
    margin: {
      top: 20,
      left: 10
    },
    fontsize: 20
  },

  colorExtent: [
    d3.rgb('#fce94f'),
    d3.rgb('#cc0001')
  ]
};

var month = ["Jan","Feb","Mar","Apr","May","Jun","Jul","Aug","Sept","Oct","Nov","Dec"];

// Visualization setup
var visDiv = d3.select('#d3l'),
  visSvg = visDiv.append('svg'),
  visGrp = visSvg.append('g'),
  grpYear = visSvg.append('g'),
  rectYear = grpYear.append('rect'),
  txtYear = grpYear.append('text'),
  grpPoints = visSvg.append('g'),
  grpTip = visSvg.append('g');

// D3 Visualization Layer
function D3Layer() {

  var layer = {},
    bounds,
    feature,
    collection,
    firstDraw = true,
    magExtent,
    dayExtent,
    eqRadius,
    eqDelay,
    eqDuration,
    eqColor;

  var startDay = 0;
  var lastMax = 0;

  layer.parent = visDiv.node();

  layer.project = function(coord) {
    var svgPoint = layer.map.locationPoint({ lat: coord[1], lon: coord[0] });
    return [svgPoint.x, svgPoint.y];
  };

  layer.draw = function() {

    if (firstDraw) {

      var mapDim = layer.map.dimensions,
      btnPlay = d3.select('#btnPlay')
        .attr('disabled', null)
        .on('click', layer.drawPoints);

      visSvg.attr('width',  mapDim.x)
        .attr('height', mapDim.y);
      /*
      // Bottom right
      var infoPos = {
      x: mapDim.x - visconf.rectYear.width - visconf.rectYear.margin,
      y: mapDim.y - visconf.rectYear.height - visconf.rectYear.margin
      };
      */

      // Top right
      var infoPos = {
        x: mapDim.x - visconf.rectYear.width - visconf.rectYear.margin,
        y: 0 + visconf.rectYear.margin
      };

      grpYear.attr("transform", "translate(" + infoPos.x + "," + infoPos.y + ")");

      rectYear.attr('id', 'infobox')
        .attr('x', 0)
        .attr('y', 0)
        .attr('width',  visconf.rectYear.width)
        .attr('height', visconf.rectYear.height);

      txtYear.attr('id', 'txtyear')
        .style('font-size', visconf.txtYear.fontsize+'px')
        .attr('x', visconf.txtYear.margin.left)
        .attr('y', visconf.txtYear.margin.top)
        .text('Jan/2009');

      firstDraw = false;
    }

  };

  layer.drawPoints = function() {

    btnPlay = d3.select('#btnPlay').text('Resume').attr('disabled', 'disabled');

    path = d3.geo.path()
      .projection(layer.project)
      .pointRadius(0);

    feature.attr("d", path);

    path = d3.geo.path()
      .projection(layer.project)
      .pointRadius(function(item) {
        return eqRadius(item.properties.mag);
      });

    // Very first day
    var firstDay = 0;

    // Clear points layer
    grpPoints.selectAll("*").remove();
    //grpTip.selectAll("*").remove();

    // un-Highlight matching LI
    $('.earthquake_item').removeClass('active');

    // Offset for resume filter
    startDay = lastMax + 1;

    // Reset last maximum magnitude
    lastMax = 0;

    feature.filter(function(d, i) {
      if (d.properties.day < firstDay || firstDay === 0) {
        firstDay = d.properties.day;
      }
      if ((d.properties.day <= lastMax || lastMax === 0) && d.properties.day >= startDay) {
        if (d.properties.mag >= 7) lastMax = d.properties.day;
        return i;
      }
    })
    .transition()
    .delay(function(item) {
      // Reposition to begining based on day of last resume.
      if (startDay === 1) startDay = firstDay;
      return eqDelay(item.properties.day-(startDay-firstDay));
    })
    .duration(function(item) {
      return eqDuration(item.properties.mag);
    })
    .each('start', function() {
      var mag = this.__data__.properties.mag;

      d3.select(this)
        .attr('fill', function() {
          return eqColor(Math.floor(mag));
        })
        .attr('fill-opacity', 0.2);

      txtYear.text(this.__data__.properties.month+'/'+this.__data__.properties.year);
    })
    .each('end', function() {

      d3.select(this).attr("fill-opacity", 0.0);

      var mag = this.__data__.properties.mag;
      if (mag >= 7) {

        // Turn radius back on
        d3.select(this)
          .attr('fill-opacity', 0.2);

        // Add to list
        var code = this.__data__.properties.code;
        var datetime = new Date(this.__data__.properties.time);
        var html_item = '<li class="list-group-item earthquake_item active" id="earthquake_'+code+'">' + datetime.toLocaleTimeString()+' '+datetime.toLocaleDateString() +
          '<br/>' + this.__data__.properties.place +
          '<br/>' + 'Magnitude: '+this.__data__.properties.mag;
        if (this.__data__.properties.dmin !== null) {
          html_item += '<br/>' + 'Depth: '+Math.round(this.__data__.properties.dmin,2)+'km';
        }
        html_item += '</li>';

        $('#earthquake_list').prepend(html_item);

        // Insert numbered circle
        var segments = this.pathSegList;
        var pointX = segments.getItem(0).x;
        var pointY = segments.getItem(0).y;

        // Save data on li for later mouseover.
        //$('#earthquake_'+code).data('svg', this);
        //$('#earthquake_'+code).data('mag', mag);
        $('#earthquake_'+code).data('x', pointX);
        $('#earthquake_'+code).data('y', pointY);
        $('#earthquake_'+code).data('fill', document.defaultView.getComputedStyle(this, null).getPropertyValue("fill"));

        var cirPoint = grpPoints.append('circle');
        //  txtPoint = grpPoints.append('text'),

        cirPoint.attr('id', 'cirPoint_'+code)
          .style("fill", document.defaultView.getComputedStyle(this, null).getPropertyValue("fill"))
          .attr('class', 'cirPoint')
          .attr('cx', pointX)
          .attr('cy', pointY)
          .attr('r', visconf.cirPoint.radius)
          .on("mouseover", function() { 
            d3.select(this)
              .style("stroke", "#fff")
              .transition()
              .duration(1000)
              .style("stroke", "#B23600");

            // Highlight matching LI
            $('#earthquake_'+code).addClass('active');

          })
          .on("mouseout", function() { 
            d3.select(this)
              // if you remove this transition, 
              // the "mouseover" transition takes precedence 
              // and leaves the border "stuck" at red
              .transition()     
              .duration(500)
              .style("stroke", "#fff");

            // un-Highlight matching LI
            $('#earthquake_'+code).removeClass('active');

          });

        btnPlay = d3.select('#btnPlay').attr('disabled', null);
      }
    })
    .attr('d', path);
  };

  layer.data = function(x) {
    collection = x,
      bounds = d3.geo.bounds(collection),
      feature = visGrp.selectAll('path')
        .data(collection.features)
        .enter()
        .append('path');

    // Compute the data extent
    magExtent = d3.extent(collection.features, function(item) {
      return item.properties.mag;
    }),
    dayExtent = d3.extent(collection.features, function(item) {
      return item.properties.day;
    });

    // Compute the delay, color, radius and duration scales
    eqRadius = d3.scale.pow()
      .domain(magExtent)
      .rangeRound(visconf.radExtent)
      .exponent(visconf.radExp),
    eqDelay = d3.scale.linear()
      .domain(dayExtent)
      .rangeRound([10, visconf.duration]),
    eqDuration = d3.scale.linear()
      .domain(magExtent)
      .rangeRound(visconf.durationEntent),
    eqColor = d3.scale.linear()
      .domain(magExtent)
      .range(visconf.colorExtent);

    return layer;
  };

  layer.extent = function() {
    return new MM.Extent(
      new MM.Location(bounds[0][1], bounds[0][0]),
      new MM.Location(bounds[1][1], bounds[1][0]));
  };

  return layer;

};

function epochDay(datetime) {
  var MS_DAY = 24 * 60 * 60 * 1000,
  ms_epoch = Date.parse(datetime);
  return (ms_epoch - ms_epoch % MS_DAY) / MS_DAY;
};

// Load the data
d3.json('data/usgs_3plus_dsc.json', function(earthquakeData) {

  // Add additional data to the eartquake events
  var earthquakePoints = earthquakeData.features,
    firstDate = new Date(earthquakePoints[0].properties.time),
    dayOffset = Math.abs(epochDay(firstDate));

  earthquakePoints.forEach(function(item) {
    var datetime = new Date(item.properties.time);
    item.properties['day'] = epochDay(datetime) + dayOffset;
    item.properties['month'] = month[datetime.getMonth()];
    item.properties['year'] = datetime.getFullYear();
  });

  // Load and draw the map
  mapbox.load(mapconf.mapid, function(mbmap) {
    map = mapbox.map("map", mbmap.layer, null, []);
    earthquakeLayer = D3Layer().data(earthquakeData);
    map.addLayer(earthquakeLayer);

    // Configure the inital state of the map
    map.setExtent(mapconf.extent);
    //map.zoom(mapconf.zoom);
    //map.ui.zoomer.add();
    map.ui.attribution.add()
      .content('<a href="http://mapbox.com/about/maps">Mapbox</a><br/><a href="http://earthquake.usgs.gov/earthquakes/search/">USGS</a>');
  });

  $('#earthquake_list').on('mouseover','.earthquake_item',function(e) {
    var cirPoint = grpPoints.append('circle');

    cirPoint.attr('id', 'cirPoint_'+e.currentTarget.id)
      .style("fill", $(e.currentTarget).data('fill'))
      .attr('class', 'cirPoint')
      .attr('cx', $(e.currentTarget).data('x'))
      .attr('cy', $(e.currentTarget).data('y'))
      .attr('r', visconf.cirPoint.radius);

    $(e.currentTarget).addClass('active');

    // Turn radius back on
    //d3.select($(e.currentTarget).data('svg'))
    //  .attr('fill-opacity', 0.2); // FIXME

  });

  $('#earthquake_list').on('mouseout','.earthquake_item',function(e) {
    d3.select('#cirPoint_'+e.currentTarget.id).remove();

    // Un-Highlight matching LI
    $(e.currentTarget).removeClass('active');
  });

});

</script>
