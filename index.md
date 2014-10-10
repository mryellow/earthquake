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
        width:  400,
        height: 70,
        margin: 40
      },
      txtYear: {
        margin: {
          top: 80,
          left: 30
        },
        fontsize: 40
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
      visSvg = visDiv.append('svg')
      visGrp = visSvg.append('g'),
      grpYear = visSvg.append('g'),
      rectYear = grpYear.append('rect'),
      txtYear = grpYear.append('text'),
      grpPoints = visSvg.append('g'),
      grpTip = visSvg.append('g');

/*
  var tip = d3.tip()
      .attr('class', 'd3-tip')
      .offset([-10, 0])
      .html(function(d) {
        return "<strong>Mag:</strong> <span style='color:red'>" + d.mag + "</span>";
      });

  var div = d3.select("body")
    .append("div")  // declare the tooltip div 
    .attr("class", "tooltip")              // apply the 'tooltip' class
    .style("opacity", 0); 
  

*/

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
        var infoPos = {
          x: mapDim.x - visconf.rectYear.width - visconf.rectYear.margin,
          y: mapDim.y - visconf.rectYear.height - visconf.rectYear.margin
        };
*/
        var infoPos = {
          x: mapDim.x + mapDim.width - visconf.rectYear.width - visconf.rectYear.margin,
          y: mapDim.y - mapDim.height + visconf.rectYear.margin
        };

        grpYear.attr("transform", "translate(" + infoPos.x + "," + infoPos.y + ")");

        rectYear.attr('id', 'infobox')
            .attr('x', 0)
            .attr('y', 0)
            .attr('width',  visconf.rectYear.width)
            .attr('height', visconf.rectYear.height);

        txtYear.attr('id', 'txtyear')
          .attr('x', visconf.txtYear.margin.left)
          .attr('y', visconf.txtYear.margin.top)
          .text('Jan/2009');
/*

          tipBox.attr('id', 'tipBox')
            .attr('x', 0)
            .attr('y', 0)

            .attr('width',  visconf.tipBox.width)
            .attr('height', visconf.tipBox.height);

          tipInfo.attr('id', 'tipInfo')
            .attr('x', visconf.tipInfo.margin.left)
            .attr('y', visconf.tipInfo.margin.top)
            .text('aaaaa');
*/
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

    var firstDay = 0;

    // Clear points layer
    grpPoints.selectAll("*").remove();
    grpTip.selectAll("*").remove();

    startDay = lastMax + 1;
    lastMax = 0;
//console.log(feature);
    feature.filter(function(d, i) {
        if (d.properties.day < firstDay || firstDay === 0) {
          firstDay = d.properties.day;
        }
        if ((d.properties.day <= lastMax || lastMax === 0) && d.properties.day >= startDay && d.properties.mag >= 7) {
          //console.log(d);
          lastMax = d.properties.day;
        }
        if ((d.properties.day <= lastMax || lastMax === 0) && d.properties.day >= startDay)  return i;
      })
      .transition()
      .delay(function(item) {
        if (startDay === 1) startDay = firstDay;
        /*
        console.log('firstDay:'+firstDay);
        console.log('startDay:'+startDay);
        console.log((item.properties.day-(startDay-firstDay)));
        */
        return eqDelay(item.properties.day-(startDay-firstDay));
      })
      .duration(function(item) {
        return eqDuration(item.properties.mag);
      })
      .each('start', function() {
        var mag = this.__data__.properties.mag;
        if (mag >= 7) {
          //console.log('start:'+mag);

//console.log(this);
/*
var segments = this.pathSegList;
var pointX = segments.getItem(0).x;
var pointY = segments.getItem(0).y;

var grpTip = visSvg.append('g'),
  tipBox = grpTip.append('rect'),
  tipInfo = grpTip.append('text');

tipBox.attr('id', 'tipBox_'+this.__data__.properties.code)
  .attr('class', 'tipBox')
  .attr('x', pointX)
  .attr('y', pointY)
  .attr('width',  visconf.tipBox.width)
  .attr('height', visconf.tipBox.height);

tipInfo.attr('id', 'tipInfo_'+this.__data__.properties.code)
  .attr('class', 'tipInfo')
  .attr('x', pointX+visconf.tipInfo.margin.left)
  .attr('y', pointY+visconf.tipInfo.margin.top)
  .text('Mag:'+this.__data__.properties.mag);
*/

//console.log(this.getBBox());
/*
          div.transition()
            .duration(500)  
            .style("opacity", 0);
          div.transition()
            .duration(200)  
            .style("opacity", .9);  
          div.html(
            '<a href= "http://google.com">' + // The first <a> tag
            mag +
            "</a>")  
            .style("left", (d3.event.pageX) + "px")      
            .style("top", (d3.event.pageY - 28) + "px");
*/
          //tip.show(this.__data__.properties);
//          d3.select(this)
//            .append('tooltip')
//            .html('tooltip');
/*
          d3.select(this)
            .on('mouseover', tip.show)
            .on('mouseout', tip.hide)
*/
          //feature.transition().duration( 0 ); // pause

          // Remove tweens
          //this.remove();
        }

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
          // show info box?
          //console.log('end:'+mag);

//console.log(this.__data__.properties);

// Insert numbered circle
var segments = this.pathSegList;
var pointX = segments.getItem(0).x;
var pointY = segments.getItem(0).y;


var cirPoint = grpPoints.append('circle'),
  txtPoint = grpPoints.append('text'),
  rectTip = grpTip.append('rect'),
  txtTip = grpTip.append('text');

rectTip.attr('id', 'rectTip_'+this.__data__.properties.code)
  .attr('class', 'rectTip')
  .attr('x', pointX+visconf.rectTip.margin.left)
  .attr('y', pointY+visconf.rectTip.margin.top)
  .attr('width',  visconf.rectTip.width)
  .attr('height', visconf.rectTip.height);

txtTip.attr('id', 'txtTip_'+this.__data__.properties.code)
  .attr('class', 'txtTip')
  .attr('x', pointX+visconf.rectTip.margin.left+visconf.txtTip.margin.left)
  .attr('y', pointY+visconf.rectTip.margin.top+visconf.txtTip.margin.top);
  /*
  txtTip.append('tspan')
    .attr('dy', 1)
    .text('Title:'+this.__data__.properties.title);
  */


  var datetime = new Date(this.__data__.properties.time);

  txtTip.append('tspan')
    .text(datetime.toLocaleTimeString()+' '+datetime.toLocaleDateString());

  txtTip.append('tspan')
    .attr('x', txtTip.attr('x'))
    .attr('dy', 22)
    .text(this.__data__.properties.place);

  txtTip.append('tspan')
    .attr('x', txtTip.attr('x'))
    .attr('dy', 22)
    .text('Magnitude: '+this.__data__.properties.mag);

  if (this.__data__.properties.dmin !== null) {
    txtTip.append('tspan')
      .attr('x', txtTip.attr('x'))
      .attr('dy', 22)
      .text('Depth: '+this.__data__.properties.dmin+'km');
  }





cirPoint.attr('id', 'cirPoint_'+this.__data__.properties.code)
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

      rectTip
        .style("opacity", 0)
        .transition()
        .duration(1000)
        .style("opacity", 1);

      txtTip
        .style("opacity", 0)
        .transition()
        .duration(1000)
        .style("opacity", 1);

    })
    .on("mouseout", function() { 
       d3.select(this)
         // if you remove this transition, 
         // the "mouseover" transition takes precedence 
         // and leaves the border "stuck" at red
         .transition()     
         .duration(500)
        .style("stroke", "#fff");

      rectTip
        .style("opacity", 1)
        .transition()
        .duration(1000)
        .style("opacity", 0);

      txtTip
        .style("opacity", 1)
        .transition()
        .duration(1000)
        .style("opacity", 0);

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

      //console.log(item.properties['day']+'/'+item.properties['year']);

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
        .content('<a href="http://mapbox.com/about/maps">Mapbox</a>');
    });

  });

</script>
