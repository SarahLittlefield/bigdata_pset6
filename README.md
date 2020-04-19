# bigdata_pset6
<html lang='eng'>
    <head>
        <title>
            Mapping with D3
        </title>
        <script src = "https://d3js.org/d3.v5.min.js" charset="utf-8"></script>
        <script src="https://d3js.org/d3-scale-chromatic.v1.min.js"></script>
        <script src="https://d3js.org/topojson.v2.min.js"></script>
        <script src="https://d3js.org/d3-queue.v2.min.js"></script>
        <!-- add a style here -->
<style>

path {
        stroke: #fff;
        stroke-width: .5px;
      }
      path:hover {
        stroke-width: 2px;
      }
      body {
        text-align: center;
      }
      .tooltip {
          position: absolute;
          text-align: center;
          width: 160px;
          height: 40px;
          padding: 5px;
          font: 12px sans-serif;
          background: white;
          opacity: 0.6;
          border: 0px;
          pointer-events: none;
      }

</style>
    </head>
    <body>

      <div id='barchart'></div>
      <div id='map'></div>

    	<!-- Page elements and content go here. -->
    	<script>
    		// Our D3 code will go here.
        //3. Create SVG Canvas
        var agriculturedata = [["Brazil", 33.9],["Canada", 6.9],["Costa Rica", 34.5],["Denmark", 62],["Fiji", 23.3],["France", 52.4],["Greenland", .6],["Italy", 43.2],["Mali",33.8],["Netherlands",53.3]];

        var width= 850
        var height = 580
        var barPadding = 1

        // Get length of dataset
        var arrayLength = agriculturedata.length; // length of dataset
        var maxValue = d3.max(agriculturedata, function(d) { return +d;} ); // get max value of our dataset
        var x_axisLength = 100; // length of x-axis in our layout
        var y_axisLength = 100; // length of y-axis in our layout

        //////////////////////////////////////
        ///chart
        /////////////////////////////////////

  			var svg = d3.select("#barchart").append("svg")
  						.attr("width", width)
  						.attr("height", height);

  			svg.selectAll("rect")
                 .data(agriculturedata)
                 .enter()
                 .append("rect")
                 .attr("y", function(d, i) {
                         return i*30 + 30;
                    })
                 .attr("x", 10)
                 .attr("width", function(d) {
                         return (d[1]*4) - barPadding;
                    })
            	   .attr("height", 20)
                 .attr("fill", "green")
  							 .attr("opacity","0.7")
                 .style("padding","3px")
                 .attr("transform", "translate(110, 50)")

  			var group = svg.append("g")
  							;

        //////////////////////////////////////
        //MAP
        /////////////////////////////////////
        var albersProjection = d3.geoAlbers()
            .scale(190000)
            .rotate([71.057,0])
            .center([0,42.313])
            .translate([width/2, height/2])

        var path = d3.geoPath()
            .projection(albersProjection)

        var svg = d3.select("body").append("svg")
                    .attr("width",width)
                    .attr("height",height)

              var data311= d3.map();

              var x = d3.scaleLinear()
                        .domain([1, 8])
                        .range([600, 860]);

              //color
                    var color = d3.scaleThreshold()
                    .domain(d3.range(0,10))
                    .range(d3.schemePurples[9]);
            //legend
                    var g = svg.append("g")
                        .attr("class", "key")
                        .attr("transform", "translate(60,40)");

                    g.selectAll("rect")
                        .data(color.range().map(function(d) {
                            d = color.invertExtent(d);
                            if (d[0] == null) d[0] = x.domain()[0];
                            if (d[1] == null) d[1] = x.domain()[1];
                            return d;
                        }))
                        .enter().append("rect")
                        .attr("height", 9)
                        .attr("x", function(d) { return x(d[0]); })
                        .attr("width", function(d) { return x(d[1]) - x(d[0]); })
                        .attr("fill", function(d) { return color(d[0]); });

                    g.append("text")
                        .attr("class", "caption")
                        .attr("x", x.range()[0])
                        .attr("y", -5)
                        .attr("fill", "#000")
                        .attr("text-anchor", "start")
                        .attr("font-weight", "bold")
                        .text("Tweets per Neighborhood (log scale)");

                    g.call(d3.axisBottom(x)
                    .tickSize(13)
                    .tickFormat(function(x, i) { return i ? x : x; })
                    .tickValues(color.domain()))
                    .select(".domain")
                    .remove();

// tool tip

            var tooltip = d3.select("body").append("div")
                .attr("class", "tooltip")
                .style("opacity", 0.6)
                .style("position", "absolute")
                .style("visibility", "visible")
                .style("background", "white")
                .text(" ");


            //Load the data:
            var bosNeighborurl1='https://gist.githubusercontent.com/cesandoval/09b2e39263c748fbcb84b927cecc7c46/raw/ab71d3638efd2545ec99c2651c6f2ddcea9d2a07/boston.json'
            var tweeturl = 'https://raw.githubusercontent.com/brookefzy/BigData2020/master/pset6/boston_311.csv'


            // Queue up datasets using promises

            var promises = [
                            d3.json(bosNeighborurl1),
                            d3.csv(tweeturl, function(d) { data311.set(d.id, +Math.log(d.twitter+1));})
                            ]

                Promise.all(promises).then(ready)
                function ready([boston]) {
                  console.log(boston)
                  console.log(data311)
                  console.log(tooltip)

                  //topojson
                svg.append("g")
                .selectAll("path")
                .data(topojson.feature(boston, boston.objects.boston_neigh).features)
                .enter().append("path")
                .attr("fill", function(d) {
                  return color(d.twitter = data311.get(d.properties.OBJECTID)); })
                .attr("d",path)
                    .style("width", function(d) { return x(d) + "px"; })
                    .text(function(d) { return d; })
                    .on("mouseover", function(d) {
              tooltip.transition()
              .duration(200)
              .style("opacity", .9);
              tooltip.html(d.properties.Name + ":" + data311.get(d.properties.OBJECTID))
              .style("left", (d3.event.pageX) + "px")
              .style("top", (d3.event.pageY - 28) + "px");
            })
            .on("mouseout", function(d) {
              tooltip.transition()
              .duration(500)
              .style("opacity", 0);
            });
    ;

}
        </script>
    </body>
</html>
