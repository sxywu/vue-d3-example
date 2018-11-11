<template>
  <svg class='areachart' :width='width' :height='height'>
  </svg>
</template>

<script>
import _ from 'lodash'
import * as d3 from 'd3'

const width = 1000;
const height = 300;
const margin = {top: 0, right: 0, bottom: 20, left: 60};

export default {
  name: 'areachart',
  props: ['movies', 'filtered'],
  data() {
    return {
      width, height, margin,
    }
  },
  created() {
    this.xScale = d3.scaleLinear().range([margin.left, width - margin.right])
    this.yScale = d3.scaleLinear().range([height - margin.bottom, margin.top])
    this.colorScale = d3.scaleSequential(d3.interpolateViridis)

    this.xAxis = d3.axisBottom().scale(this.xScale).tickFormat(this.format)
  },
  watch: {
    movies: function() {
      this.calculateScales()
    },
    filtered: function() {
      this.calculateData()
    }
  },
  methods: {
    calculateScales: function() {
      if (!this.movies.length) return
    },
    calculateData: function() {
      if (!this.filtered.length) return
    }
  }
}
</script>

<style>
</style>
