<template>
    <div id="app">
        <ul id="menu">
            <li data-menuanchor="page1" class="active"><a href="#page1">Section 1</a></li>
            <li data-menuanchor="page2"><a href="#page2">Section 2</a></li>
            <li data-menuanchor="page3"><a href="#page3">Section 3</a></li>
        </ul>
        <full-page :options="options" id="fullpage">
            <div class="section">
                <img src="/assets/img/gondor_tree.svg">
                <h3>The <span class="span-lotr">Lord of the Rings</span><br /> Fanfiction Ships <span class="span-lotr">Network</span></h3>
                <div id="curly-brace">
                  <div id="left" class="brace"></div>
                  <div id="right" class="brace"></div>
                </div>
            </div>
            <div class="section">
                <div class="container">
                    <svg width="1000" height="800"></svg>
                </div>
            </div>
            <div class="section">
                <div class="slide">
                    <h3>Slide 2.1</h3>
                </div>
                <div class="slide">
                    <h3>Slide 2.2</h3>
                </div>
                <div class="slide">
                    <h3>Slide 2.3</h3>
                </div>
            </div>
        </full-page>
    </div>
</template>

<script>
import * as d3 from "d3";
export default {
  name: "app",
  data() {
    return {
      options: {
        afterLoad: this.afterLoad,
        scrollBar: false,
        menu: "#menu",
        navigation: true,
        anchors: ["page1", "page2", "page3"],
        sectionsColor: [
          "#516E9C",
          "#343745",
          "#A5AABB",
          "#fec401",
          "#1bcee6",
          "#ee1a59",
          "#2c3e4f",
          "#ba5be9",
          "#b4b8ab"
        ]
      }
    };
  },
  mounted() {
    var tooltip = d3
      .select("body")
      .append("div")
      .attr("class", "tooltip");

    var svg = d3.select("svg"),
      width = +svg.attr("width"),
      height = +svg.attr("height");

    var defs = svg.append("svg:defs");

    var config = {
      avatar_size: 40
    };

    // getDistance = () => {
    //   return 100;
    // };

    svg
      .append("clipPath")
      .attr("id", "clipObj")
      .append("circle")
      .attr("cx", config.avatar_size / 2)
      .attr("cy", config.avatar_size / 2)
      .attr("r", config.avatar_size / 2);

    var color = d3.scaleOrdinal(d3.schemeCategory20);

    var simulation = d3
      .forceSimulation()
      .force(
        "link",
        d3
          .forceLink()
          .strength(0.1)
          .distance(100)
          .id(function(d) {
            return d.id;
          })
      )
      //.force("link", d3.forceLink().distance(linkDistance).strength(0.1))
      .force("charge", d3.forceManyBody())
      .force("center", d3.forceCenter(width / 2, height / 2));

    d3.json("graphFile.json", function(error, graph) {
      if (error) throw error;

      var link = svg
        .append("g")
        .attr("class", "links")
        .selectAll("line")
        .data(graph.links)
        .enter()
        .append("line")
        .attr("stroke-width", function(d) {
          console.log(d);
          return d.value;
        })
        .style("stroke", "#fff")
        .on("mouseover", function(d, i, a) {
          console.log(this);
          d3
            .select(this)
            .transition()
            .style("stroke", "red");
          //.attr("height", config.avatar_size + 24);
          return tooltip
            .style("visibility", "visible")
            .text(`Wow so many ${d.value}`);
          // .style("top", event.pageY - 30 + "px")
          // .style("left", event.pageX + "px");
        })
        // .on("mousemove", function() {
        //     return tooltip.style("top", (event.pageY - 30) + "px")
        //         .style("left", event.pageX + "px");
        // })
        .on("mouseout", function() {
          d3
            .select(this)
            .transition()
            .style("stroke", "#fff");
          // .attr("width", config.avatar_size)
          // .attr("height", config.avatar_size);
          return tooltip.style("visibility", "hidden");
        });

      var circle = svg
        .append("g")
        .attr("class", "circles")
        .selectAll("circle")
        .data(graph.nodes)
        .enter()
        .append("circle")
        // .attr('x', 20)
        // .attr('y', 20)
        .attr("cx", function(d) {
          return 20;
        })
        .attr("cy", function(d) {
          return 20;
        })
        .attr("r", 35)
        .style("fill", "fdf")
        .style("stroke", "#ccc");

      var textPath = svg
        .append("g")
        .attr("class", "textPaths")
        .selectAll("path")
        .data(graph.nodes)
        .enter()
        .append("path")
        .attr("id", function(d) {
          return d.id;
        }) //Unique id of the path
        .attr("d", "M-8,30 a24,17 0 0,0 59,0") //SVG path
        .style("fill", "none")
        //.style("stroke", "#fff");

      var pathText = svg
        .append("g")
        .attr("class", "pathTexts")
        .selectAll("pathText")
        .data(graph.nodes)
        .enter()
        .append("text")
        .append("textPath") //append a textPath to the text element
        .attr("xlink:href", function(d) {
          return "#" + d.id;
        }) //place the ID of the path here
        .style("text-anchor", "middle") //place the text halfway on the arc
        .attr("startOffset", "50%")
        .text(function(d) {
          return d.name || d.id;
        })

      var node = svg
        .append("g")
        .attr("class", "nodes")
        .selectAll("image")
        .data(graph.nodes)
        .enter()
        .append("image")
        .attr("xlink:href", function(d) {
          return d.image;
        })
        .attr("x", 0)
        .attr("y", 0)
        .attr("width", config.avatar_size)
        .attr("height", config.avatar_size)
        // .attr('transform', function(d) {
        //     return 'translate(' + parseInt(d.posx + config.avatar_size / 2) + ',' + parseInt(d.posy +
        //         config.avatar_size / 2) + ')'
        // })
        .attr("clip-path", "url(#clipObj)")
        .call(
          d3
            .drag()
            .on("start", dragstarted)
            .on("drag", dragged)
            .on("end", dragended)
        );

      node.append("title").text(function(d) {
        return d.id;
      });

      // optional labels
      // var label = svg.append("g")
      //     .attr("class", "labels")
      //     .selectAll("text")
      //     .data(graph.nodes)
      //     .enter().append("text")
      //     .attr("class", "label")
      //     .text(function(d) {
      //         return d.id;
      //     })

      simulation.nodes(graph.nodes).on("tick", ticked);

      simulation.force("link").links(graph.links);

      function ticked() {
        link
          .attr("x1", function(d) {
            return d.source.x + 20;
          })
          .attr("y1", function(d) {
            return d.source.y + 20;
          })
          .attr("x2", function(d) {
            return d.target.x + 20;
          })
          .attr("y2", function(d) {
            return d.target.y + 20;
          });

        node.attr("transform", function(d) {
          return "translate(" + d.x + "," + d.y + ")";
        });

        circle.attr("transform", function(d) {
          return "translate(" + d.x + "," + d.y + ")";
        });

        textPath.attr("transform", function(d) {
          return "translate(" + d.x + "," + d.y + ")";
        });

        pathText.attr("transform", function(d) {
          return "translate(" + d.x + "," + d.y + ")";
        });

        // label
        //     .attr("x", function(d) {
        //         return d.x;
        //     })
        //     .attr("y", function(d) {
        //         return d.y;
        //     })
        //     .style("font-size", "12px").style("font-family", "Arial").style("fill", "#333");
      }
    });

    function dragstarted(d) {
      if (!d3.event.active) simulation.alphaTarget(0.3).restart();
      d.fx = d.x;
      d.fy = d.y;
    }

    function dragged(d) {
      d.fx = d3.event.x;
      d.fy = d3.event.y;
    }

    function dragended(d) {
      if (!d3.event.active) simulation.alphaTarget(0);
      d.fx = null;
      d.fy = null;
    }
  },
  methods: {
    afterLoad() {
      console.log("After load");
    }
  }
};
</script>

<style>
body {
  font-family: "Poppins", Sans-Serif;
  background-image: url("../assets/img/bg.jpg");
  color: #fff;
}

#app {
  opacity: 0.92;
}

ul {
  list-style-type: none;
  padding: 0;
}

li {
  display: inline-block;
  margin: 0 10px;
}

h3 {
  font-family: "Poppins", Sans-Serif;
  font-size: 34px;
}

.span-lotr {
  font-family: "Abril Fatface", Cursive;
}

.pathTexts {
  font-size: 12px;
}

.links line {
  stroke-opacity: 1;
}

.tooltip {
  position: absolute;
  z-index: 10;
  visibility: hidden;
  text-align: center;
  padding: 2px 20px 2px 20px;
  border-radius: 4px;
  font-weight: bold;
  color: #000;
  -webkit-box-sizing: content-box;
  -moz-box-sizing: content-box;
  box-sizing: content-box;
  border: none;
  font: normal normal 400 16px / normal "Abril Fatface", Helvetica, sans-serif;
  color: #343745;
  text-align: center;
  -o-text-overflow: clip;
  text-overflow: clip;
  background: #fff;
}

.container {
  width: 100%;
  text-align: center;
}

a {
  color: #42b983;
}

#curly-brace {
  width: 400px;
  position: relative;
  left: 50%;
  margin-left: -200px;
  top: 20px;
}

.brace {
  border-bottom: 5px solid;
  margin: 0 10%;
  width: 30%;
  height: 25px;
  float: left;
}

.brace:before,
.brace:after {
  content: "";
  width: 10%;
  height: 50px;
  position: absolute;
  display: block;
}

#left:before {
  border-bottom: 5px solid;
  border-bottom-left-radius: 20px;
  margin-left: -10%;
  margin-top: -30px;
}

#left:after {
  border-top: 5px solid;
  border-top-right-radius: 20px;
  margin-left: 30%;
  margin-top: 20px;
}

#right:before {
  border-top: 5px solid;
  border-top-left-radius: 20px;
  margin-left: -10%;
  margin-top: 20px;
}

#right:after {
  border-bottom: 5px solid;
  border-bottom-right-radius: 20px;
  margin-left: 30%;
  margin-top: -30px;
}
</style>
