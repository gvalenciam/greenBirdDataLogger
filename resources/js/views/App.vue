<template>
  <v-app>
    <v-card>
      <v-toolbar color="#4D6B87" dark flat>
        <v-app-bar-nav-icon></v-app-bar-nav-icon>

        <v-toolbar-title>Data Logger</v-toolbar-title>

        <template v-slot:extension>
          <v-tabs v-model="tab" centered background-color="transparent">
            <v-tabs-slider color="white"></v-tabs-slider>

            <v-tab v-for="item in items" :key="item.id">
              <v-icon v-if="item.id == 1" class="mx-2">mdi-database</v-icon>
              <v-icon v-else-if="item.id == 2" class="mx-2">mdi-chart-bell-curve-cumulative</v-icon>
              {{ item.name }}
            </v-tab>
          </v-tabs>
        </template>
      </v-toolbar>

      <v-tabs-items v-model="tab">
        <v-tab-item v-for="item in items" :key="item.id">
          <v-card flat>
            <v-card-text v-text="text"></v-card-text>
            <v-btn rounded class="text-none" v-on:click="testServer">Refresh</v-btn>
            <v-container fluid>
              <v-layout justify-center align-center>
                <div id="csvGraph"></div>
              </v-layout>
            </v-container>
          </v-card>
        </v-tab-item>
      </v-tabs-items>
    </v-card>
  </v-app>
</template>

<script>
// import * as d3 from 'd3';

export default {
  data() {
    return {
      graphData: [],
      idleTimeout: 0,
      tab: null,
      items: [
        {
          id: 1,
          name: 'Señales'
        },
        {
          id: 2,
          name: 'Imágenes'
        }
      ],
      text:
        'Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat.'
    };
  },
  methods: {
    refreshGraph: function() {
      var contThis = this;

      var margin = { top: 10, right: 30, bottom: 30, left: 30 };
      var width = window.screen.width * 0.5 - margin.left - margin.right;
      var height = window.screen.height / 2 - margin.top - margin.bottom;
      var yMin = 0;
      var yMax = 10;
      var xMin = 0;
      var xMax = 10;

      var svg = d3
        .select('#csvGraph')
        .append('svg')
        .attr('width', width + margin.left + margin.right)
        .attr('height', height + margin.top + margin.bottom)
        .append('g')
        .attr('id', 'maing')
        .attr('transform', 'translate(' + margin.left + ',' + margin.top + ')');

      d3.csv('data/test.csv', function(data) {
        console.log(data);

        // Add X axis
        var x = d3
          .scaleLinear()
          .domain([xMin, xMax])
          .range([0, width]);
        var xAxis = svg
          .append('g')
          .attr('transform', 'translate(0,' + height + ')')
          .call(d3.axisBottom(x));

        // Add Y axis
        var y = d3
          .scaleLinear()
          .domain([yMin, yMax])
          .range([height, 0]);
        svg.append('g').call(d3.axisLeft(y));

        // Add a clipPath: everything out of this area won't be drawn.
        var clip = svg
          .append('defs')
          .append('svg:clipPath')
          .attr('id', 'clip')
          .append('svg:rect')
          .attr('width', width)
          .attr('height', height)
          .attr('x', 0)
          .attr('y', 0);

        // Brushing
        var brush = d3
          .brushX()
          .extent([[0, 0], [width, height]])
          .on('end', updateChart);

        var scatter = svg.append('g').attr('clip-path', 'url(#clip)');

        scatter
          .selectAll('circle')
          .data(data)
          .enter()
          .append('circle')
          .attr('cx', function(d) {
            return x(d.valuex);
          })
          .attr('cy', function(d) {
            return y(d.valuey);
          })
          .attr('r', 3)
          .style('fill', '#69b3a2');

        // Brushing
        scatter
          .append('g')
          .attr('class', 'brush')
          .call(brush);

        // Function idleTimeout null
        var idleTimeout;
        function idled() {
          idleTimeout = null;
        }

        // Update chart for given boundaries
        function updateChart() {
          var extent = d3.event.selection;

          if (!extent) {
            if (!idleTimeout) return (idleTimeout = setTimeout(idled, 350));
            x.domain([xMin, xMax]);
          } else {
            x.domain([x.invert(extent[0]), x.invert(extent[1])]);
            scatter.select('.brush').call(brush.move, null);
          }

          // Update axis and circles position
          xAxis
            .transition()
            .duration(1000)
            .call(d3.axisBottom(x));
          scatter
            .selectAll('circle')
            .transition()
            .duration(1000)
            .attr('cx', function(d) {
              return x(d.valuex);
            })
            .attr('cy', function(d) {
              return y(d.valuey);
            });
        }

        // Add dots
      });
    },
    testServer: function() {
      axios('http://localhost:80', {
        method: 'GET',
        mode: 'no-cors',
        headers: {
          'Access-Control-Allow-Origin': '*',
          'Content-Type': 'text/html; charset=utf-8'
        },
        withCredentials: false,
        credentials: 'same-origin'
      }).then(response => console.log(response));
    }
  }
};
</script>

<style>
</style>