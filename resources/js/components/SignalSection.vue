<template>
  <span>
    <v-container fluid>
      <v-layout class="sectionContent py-6">
        <!-- <div id="csvGraph" class="csvGraph ml-12"></div> -->
        <svg id="csvGraph" class="csvGraph" width="700" height="500">
          <svg:style>.line{fill: none; stroke: steelblue; overflow:hidden; stroke-width: 1.5px;} .zoom{fill: none; stroke: steelblue; overflow: hidden; cursor:move; pointer-events: all}</svg:style>
        </svg>
        <div id="selectContainer" class="selectContainer mr-12">
          <v-btn
            rounded
            depressed
            class="text-none white--text mb-6"
            color="#4C759B"
            v-on:click="refreshGraph"
          >Refrescar</v-btn>
          <v-select
            :key="selectKey"
            v-on:change="selectGraphUpdate"
            multiple
            small-chips
            deletable-chips
            persistent-hint
            label="Gráfico"
            hint="Seleccione los gráficos a mostrar"
            color="#4C759B"
            item-color="#4C759B"
            :items="this.selectItems"
          ></v-select>
        </div>
      </v-layout>
    </v-container>
  </span>
</template>

<script>
export default {
  name: 'SignalSection',
  mounted() {
    this.refreshGraph();
    // this.brushScrollZoom();
  },
  data() {
    return {
      graphData: [],
      selectItems: [
        {
          text: 'Contenido Volumetrico agua (m3/m3 %) @ -30cm',
          value: 'Contenido_agua'
        },
        { text: 'Potencial matricial @-30cm', value: 'Potencial' },
        { text: 'Temperatura ˚C @-30cm', value: 'Temperatura' }
      ],
      csvColumns: [],
      selectCheckedItems: [],
      selectKey: 1
    };
  },
  computed: {},
  methods: {
    // refreshGraph: function() {
    //   var contThis = this;
    //   this.forceRender();
    //   this.selectCheckedItems = [];

    //   $('#csvGraph').empty();

    //   var margin = { top: 10, right: 30, bottom: 30, left: 30 };
    //   var width = window.screen.width * 0.5 - margin.left - margin.right;
    //   var height = window.screen.height * 0.6 - margin.top - margin.bottom;
    //   var yMin = 0;
    //   var yMax = 20;
    //   var xMin = 0;
    //   var xMax = 12;

    //   var svg = d3
    //     .select('#csvGraph')
    //     .append('svg')
    //     .attr('width', width + margin.left + margin.right)
    //     .attr('height', height + margin.top + margin.bottom)
    //     .append('g')
    //     .attr('id', 'maing')
    //     .attr('transform', 'translate(' + margin.left + ',' + margin.top + ')');

    //   d3.csv('data/test.csv', function(data) {
    //     var columns = data.columns;
    //     columns.shift();
    //     contThis.csvColumns = columns;

    //     var allGroup = columns;

    //     var dataReady = allGroup.map(function(grpName) {
    //       return {
    //         name: grpName,
    //         values: data.map(function(d) {
    //           return { time: d.time, value: +d[grpName] };
    //         })
    //       };
    //     });

    //     var myColor = d3
    //       .scaleOrdinal()
    //       .domain(allGroup)
    //       .range(d3.schemeSet2);

    //     var x = d3
    //       .scaleLinear()
    //       .domain([0, xMax])
    //       .range([0, width]);
    //     svg
    //       .append('g')
    //       .attr('transform', 'translate(0,' + height + ')')
    //       .call(d3.axisBottom(x));

    //     // Add Y axis
    //     var y = d3
    //       .scaleLinear()
    //       .domain([0, yMax])
    //       .range([height, 0]);
    //     svg.append('g').call(d3.axisLeft(y));

    //     // Add the lines
    //     var line = d3
    //       .line()
    //       .x(function(d) {
    //         return x(+d.time);
    //       })
    //       .y(function(d) {
    //         return y(+d.value);
    //       });
    //     svg
    //       .selectAll('myLines')
    //       .data(dataReady)
    //       .enter()
    //       .append('path')
    //       .attr('class', function(d) {
    //         return d.name;
    //       })
    //       .attr('d', function(d) {
    //         return line(d.values);
    //       })
    //       .attr('stroke', function(d) {
    //         return myColor(d.name);
    //       })
    //       .style('opacity', 0)
    //       .style('stroke-width', 4)
    //       .style('fill', 'none');

    //     // Add the points
    //     svg
    //       // First we need to enter in a group
    //       .selectAll('myDots')
    //       .data(dataReady)
    //       .enter()
    //       .append('g')
    //       .style('fill', function(d) {
    //         return myColor(d.name);
    //       })
    //       .attr('class', function(d) {
    //         return d.name;
    //       })
    //       .style('opacity', 0)
    //       // Second we need to enter in the 'values' part of this group
    //       .selectAll('myPoints')
    //       .data(function(d) {
    //         return d.values;
    //       })
    //       .enter()
    //       .append('circle')
    //       .attr('cx', function(d) {
    //         return x(d.time);
    //       })
    //       .attr('cy', function(d) {
    //         return y(d.value);
    //       })
    //       .attr('r', 5)
    //       .attr('stroke', 'white');

    //     // Add a label at the end of each line
    //     svg
    //       .selectAll('myLabels')
    //       .data(dataReady)
    //       .enter()
    //       .append('g')
    //       .append('text')
    //       .attr('class', function(d) {
    //         return d.name;
    //       })
    //       .datum(function(d) {
    //         return { name: d.name, value: d.values[d.values.length - 1] };
    //       }) // keep only the last value of each time series
    //       .attr('transform', function(d) {
    //         return (
    //           'translate(' + x(d.value.time) + ',' + y(d.value.value) + ')'
    //         );
    //       }) // Put the text at the position of the last point
    //       .attr('x', 12) // shift the text a bit more right
    //       .text(function(d) {
    //         return d.name;
    //       })
    //       .style('fill', function(d) {
    //         return myColor(d.name);
    //       })
    //       .style('font-size', 15)
    //       .style('opacity', 0);
    //   });
    // },
    refreshGraph: function() {
      var contThis = this;
      this.forceRender();
      this.selectCheckedItems = [];

      $('#csvGraph').empty();

      var margin = { top: 10, right: 30, bottom: 30, left: 30 };
      var width = window.screen.width * 0.5 - margin.left - margin.right;
      var height = window.screen.height * 0.55 - margin.top - margin.bottom;
      var yMin = 0;
      var yMax = 20;
      var xMin = 0;
      var xMax = 12;

      var svg = d3
        .select('#csvGraph')
        .append('svg')
        .attr('width', width + margin.left + margin.right)
        .attr('height', height + margin.top + margin.bottom)
        .append('g')
        .attr('id', 'maing')
        .attr('transform', 'translate(' + margin.left + ',' + margin.top + ')');

      d3.csv('data/test2.csv', function(data) {
        var columns = data.columns;
        columns.shift();
        contThis.csvColumns = columns;

        var allGroup = columns;

        var dataReady = allGroup.map(function(grpName) {
          return {
            name: grpName,
            values: data.map(function(d) {
              return { time: d.time, value: +d[grpName] };
            })
          };
        });

        var myColor = d3
          .scaleOrdinal()
          .domain(allGroup)
          .range(d3.schemeSet2);

        var x = d3
          .scaleLinear()
          .domain([0, xMax])
          .range([0, width]);
        svg
          .append('g')
          .attr('transform', 'translate(0,' + height + ')')
          .call(d3.axisBottom(x));

        // Add Y axis
        var y = d3
          .scaleLinear()
          .domain([0, yMax])
          .range([height, 0]);
        svg.append('g').call(d3.axisLeft(y));

        // Add the lines
        var line = d3
          .line()
          .x(function(d) {
            return x(+d.time);
          })
          .y(function(d) {
            return y(+d.value);
          });
        svg
          .selectAll('myLines')
          .data(dataReady)
          .enter()
          .append('path')
          .attr('class', function(d) {
            return d.name;
          })
          .attr('d', function(d) {
            return line(d.values);
          })
          .attr('stroke', function(d) {
            return myColor(d.name);
          })
          .style('opacity', 0)
          .style('stroke-width', 2)
          .style('fill', 'none');

        // Add the points
        svg
          // First we need to enter in a group
          .selectAll('myDots')
          .data(dataReady)
          .enter()
          .append('g')
          .style('fill', function(d) {
            return myColor(d.name);
          })
          .attr('class', function(d) {
            return d.name;
          })
          .style('opacity', 0)
          // Second we need to enter in the 'values' part of this group
          .selectAll('myPoints')
          .data(function(d) {
            return d.values;
          })
          .enter()
          .append('circle')
          .attr('cx', function(d) {
            return x(d.time);
          })
          .attr('cy', function(d) {
            return y(d.value);
          })
          .attr('r', 5)
          .attr('stroke', 'white');

        // Add a label at the end of each line
        svg
          .selectAll('myLabels')
          .data(dataReady)
          .enter()
          .append('g')
          .append('text')
          .attr('class', function(d) {
            return d.name;
          })
          .datum(function(d) {
            return { name: d.name, value: d.values[d.values.length - 1] };
          }) // keep only the last value of each time series
          .attr('transform', function(d) {
            return (
              'translate(' + x(d.value.time) + ',' + y(d.value.value) + ')'
            );
          }) // Put the text at the position of the last point
          .attr('x', 12) // shift the text a bit more right
          .text(function(d) {
            return d.name;
          })
          .style('fill', function(d) {
            return myColor(d.name);
          })
          .style('font-size', 15)
          .style('opacity', 0);
      });
    },
    selectGraphUpdate: function(data) {
      var difference = data.filter(x => !this.selectCheckedItems.includes(x));
      if (difference.length === 0) {
        difference = this.selectCheckedItems.filter(x => !data.includes(x));
      }
      this.selectCheckedItems = [...data];

      var currentOpacity = d3.selectAll('.' + difference[0]).style('opacity');
      // Change the opacity: from 0 to 1 or from 1 to 0
      d3.selectAll('.' + difference[0])
        .transition()
        .style('opacity', currentOpacity == 1 ? 0 : 1);
    },
    forceRender() {
      this.selectKey += 1;
    },
    brushScrollZoom: function() {
      var svg = d3.select('#csvGraph'),
        margin = { top: 20, right: 20, bottom: 110, left: 40 },
        margin2 = { top: 430, right: 20, bottom: 30, left: 40 },
        width = window.screen.width * 0.5 - margin.left - margin.right,
        height = window.screen.height * 0.5 - margin.top - margin.bottom,
        height2 = +svg.attr('height') - margin2.top - margin2.bottom;

      var parseDate = d3.timeParse('%m/%d/%Y %H:%M');

      var x = d3.scaleTime().range([0, width]),
        x2 = d3.scaleTime().range([0, width]),
        y = d3.scaleLinear().range([height, 0]),
        y2 = d3.scaleLinear().range([height2, 0]);

      var xAxis = d3.axisBottom(x),
        xAxis2 = d3.axisBottom(x2),
        yAxis = d3.axisLeft(y);

      var brush = d3
        .brushX()
        .extent([[0, 0], [width, height2]])
        .on('brush end', brushed);

      var zoom = d3
        .zoom()
        .scaleExtent([1, Infinity])
        .translateExtent([[0, 0], [width, height]])
        .extent([[0, 0], [width, height]])
        .on('zoom', zoomed);

      var line = d3
        .line()
        .x(function(d) {
          return x(d.Date);
        })
        .y(function(d) {
          return y(d.Air_Temp);
        });

      var line2 = d3
        .line()
        .x(function(d) {
          return x2(d.Date);
        })
        .y(function(d) {
          return y2(d.Air_Temp);
        });

      var clip = svg
        .append('defs')
        .append('svg:style')
        .append('svg:clipPath')
        .attr('id', 'clip')
        .append('svg:rect')
        .attr('width', width)
        .attr('height', height)
        .attr('x', 0)
        .attr('y', 0);

      var Line_chart = svg
        .append('g')
        .attr('class', 'focus')
        .attr('transform', 'translate(' + margin.left + ',' + margin.top + ')')
        .attr('clip-path', "url('#clip')");

      var focus = svg
        .append('g')
        .attr('class', 'focus')
        .attr('transform', 'translate(' + margin.left + ',' + margin.top + ')');

      var context = svg
        .append('g')
        .attr('class', 'context')
        .attr(
          'transform',
          'translate(' + (margin2.left - 20) + ',' + margin2.top + ')'
        );

      d3.csv('data/test3.csv', type, function(error, data) {
        if (error) throw error;

        x.domain(
          d3.extent(data, function(d) {
            return d.Date;
          })
        );
        y.domain([
          0,
          d3.max(data, function(d) {
            return d.Air_Temp;
          })
        ]);
        x2.domain(x.domain());
        y2.domain(y.domain());

        focus
          .append('g')
          .attr('class', 'axis axis--x')
          .attr('transform', 'translate(0,' + height + ')')
          .call(xAxis);

        focus
          .append('g')
          .attr('class', 'axis axis--y')
          .call(yAxis);

        Line_chart.append('path')
          .datum(data)
          .attr('class', 'line')
          .attr('d', line);

        context
          .append('path')
          .datum(data)
          .attr('class', 'line')
          .attr('d', line2);

        context
          .append('g')
          .attr('class', 'axis axis--x')
          .attr('transform', 'translate(0,' + height2 + ')')
          .call(xAxis2);

        context
          .append('g')
          .attr('class', 'brush')
          .call(brush)
          .call(brush.move, x.range());

        svg
          .append('rect')
          .attr('class', 'zoom')
          .attr('width', width)
          .attr('height', height)
          .attr(
            'transform',
            'translate(' + margin.left + ',' + margin.top + ')'
          )
          .call(zoom);

        console.log(data);
      });

      function brushed() {
        if (d3.event.sourceEvent && d3.event.sourceEvent.type === 'zoom')
          return; // ignore brush-by-zoom
        var s = d3.event.selection || x2.range();
        x.domain(s.map(x2.invert, x2));
        Line_chart.select('.line').attr('d', line);
        focus.select('.axis--x').call(xAxis);
        svg
          .select('.zoom')
          .call(
            zoom.transform,
            d3.zoomIdentity.scale(width / (s[1] - s[0])).translate(-s[0], 0)
          );
      }

      function zoomed() {
        if (d3.event.sourceEvent && d3.event.sourceEvent.type === 'brush')
          return; // ignore zoom-by-brush
        var t = d3.event.transform;
        x.domain(t.rescaleX(x2).domain());
        Line_chart.select('.line').attr('d', line);
        focus.select('.axis--x').call(xAxis);
        context.select('.brush').call(brush.move, x.range().map(t.invertX, t));
      }

      function type(d) {
        d.Date = parseDate(d.Date);
        d.Air_Temp = +d.Air_Temp;
        return d;
      }
    }
  }
};
</script>

<style scoped>
.sectionContent {
  background-color: transparent;
  display: flex;
  align-content: center;
  justify-content: space-evenly;
}
.csvGraph {
  /* width: 65%; */
  overflow: hidden;
}
.selectContainer {
  width: 35%;
}
.line {
  fill: none;
  stroke: steelblue;
  stroke-width: 1.5px;
}
.zoom {
  cursor: move;
  fill: none;
  pointer-events: all;
}
</style>