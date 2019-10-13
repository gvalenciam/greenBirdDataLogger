<template>
  <span>
    <v-container fluid>
      <v-layout class="sectionContent py-6">
        <div id="csvGraph" class="csvGraph ml-12"></div>
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
  },
  data() {
    return {
      graphData: [],
      selectItems: [
        { text: '% Agua', value: 'Agua' },
        { text: 'P. matricial', value: 'Potencial' },
        { text: 'C. eléctrica', value: 'Conductividad' }
      ],
      csvColumns: [],
      selectCheckedItems: [],
      selectKey: 1
    };
  },
  computed: {},
  methods: {
    refreshGraph: function() {
      var contThis = this;
      this.forceRender();
      this.selectCheckedItems = [];

      $('#csvGraph').empty();

      var margin = { top: 10, right: 30, bottom: 30, left: 30 };
      var width = window.screen.width * 0.5 - margin.left - margin.right;
      var height = window.screen.height * 0.6 - margin.top - margin.bottom;
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

      d3.csv('data/test.csv', function(data) {
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
          .style('stroke-width', 4)
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

        // Add a legend (interactive)
        //   svg
        //     .selectAll('myLegend')
        //     .data(dataReady)
        //     .enter()
        //     .append('g')
        //     .append('text')
        //     .attr('x', function(d, i) {
        //       return 30 + i * 60;
        //     })
        //     .attr('y', 30)
        //     .text(function(d) {
        //       return d.name;
        //     })
        //     .style('fill', function(d) {
        //       return myColor(d.name);
        //     })
        //     .style('font-size', 15)
        //     .on('click', function(d) {
        //       // is the element currently visible ?
        //       var currentOpacity = d3.selectAll('.' + d.name).style('opacity');
        //       // Change the opacity: from 0 to 1 or from 1 to 0
        //       d3.selectAll('.' + d.name)
        //         .transition()
        //         .style('opacity', currentOpacity == 1 ? 0 : 1);
        //     });
      });

      // var margin = { top: 10, right: 30, bottom: 30, left: 30 };
      // var width = window.screen.width * 0.5 - margin.left - margin.right;
      // var height = window.screen.height * 0.6 - margin.top - margin.bottom;
      // var yMin = 0;
      // var yMax = 12;
      // var xMin = 0;
      // var xMax = 12;

      // var svg = d3
      //   .select('#csvGraph')
      //   .append('svg')
      //   .attr('width', width + margin.left + margin.right)
      //   .attr('height', height + margin.top + margin.bottom)
      //   .append('g')
      //   .attr('id', 'maing')
      //   .attr('transform', 'translate(' + margin.left + ',' + margin.top + ')');

      // d3.csv('data/test.csv', function(data) {
      //   console.log(data);

      //   // Add X axis
      //   var x = d3
      //     .scaleLinear()
      //     .domain([xMin, xMax])
      //     .range([0, width]);
      //   var xAxis = svg
      //     .append('g')
      //     .attr('transform', 'translate(0,' + height + ')')
      //     .call(d3.axisBottom(x));

      //   // Add Y axis
      //   var y = d3
      //     .scaleLinear()
      //     .domain([yMin, yMax])
      //     .range([height, 0]);
      //   svg.append('g').call(d3.axisLeft(y));

      //   // Add a clipPath: everything out of this area won't be drawn.
      //   var clip = svg
      //     .append('defs')
      //     .append('svg:clipPath')
      //     .attr('id', 'clip')
      //     .append('svg:rect')
      //     .attr('width', width)
      //     .attr('height', height)
      //     .attr('x', 0)
      //     .attr('y', 0);

      //   // Brushing
      //   var brush = d3
      //     .brushX()
      //     .extent([[0, 0], [width, height]])
      //     .on('end', updateChart);

      //   var scatter = svg.append('g').attr('clip-path', 'url(#clip)');

      //   scatter
      //     .selectAll('circle')
      //     .data(data)
      //     .enter()
      //     .append('circle')
      //     .attr('cx', function(d) {
      //       return x(d.valuex);
      //     })
      //     .attr('cy', function(d) {
      //       return y(d.valuey);
      //     })
      //     .attr('r', 3)
      //     .style('fill', '#69b3a2');

      //   // Brushing
      //   scatter
      //     .append('g')
      //     .attr('class', 'brush')
      //     .call(brush);

      //   // Function idleTimeout null
      //   var idleTimeout;
      //   function idled() {
      //     idleTimeout = null;
      //   }

      //   // Update chart for given boundaries
      //   function updateChart() {
      //     var extent = d3.event.selection;

      //     if (!extent) {
      //       if (!idleTimeout) return (idleTimeout = setTimeout(idled, 350));
      //       x.domain([xMin, xMax]);
      //     } else {
      //       x.domain([x.invert(extent[0]), x.invert(extent[1])]);
      //       scatter.select('.brush').call(brush.move, null);
      //     }

      //     // Update axis and circles position
      //     xAxis
      //       .transition()
      //       .duration(1000)
      //       .call(d3.axisBottom(x));
      //     scatter
      //       .selectAll('circle')
      //       .transition()
      //       .duration(1000)
      //       .attr('cx', function(d) {
      //         return x(d.valuex);
      //       })
      //       .attr('cy', function(d) {
      //         return y(d.valuey);
      //       });
      //   }
      // });
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
    }
  }
};
</script>

<style scoped>
.sectionContent {
  display: flex;
  align-content: center;
  justify-content: center;
}
.csvGraph {
  width: 50%;
}
.selectContainer {
  width: 20%;
}
</style>