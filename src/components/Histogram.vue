<template>
  <svg class='histogram' :width='width' :height='height'>
    <g class='bars'>
      <rect v-for='(d, i) in bars' :key='i' :x='d.x' :width='d.width'
        :y='d.y' :height='d.height' :fill=d.fill :stroke='d.fill' />
    </g>
    <g ref='xAxis' :transform='`translate(0, ${height - margin.bottom})`' />
    <g ref='brush' />
  </svg>
</template>

<script>
import _ from 'lodash'
import * as d3 from 'd3'

const width = 500
const height = 120
const margin = {top: 20, right: 20, bottom: 20, left: 20}

export default {
  name: 'histogram',
  props: ['movies', 'filtered', 'id', 'format', 'updateFilters'],
  data() {
    return {
      width, height, margin,
      bars: [],
    }
  },
  created() {
    this.xScale = d3.scaleLinear().range([margin.left, width - margin.right])
    this.histogram = d3.histogram()

    this.yScale = d3.scaleLinear().range([height - margin.bottom, margin.top])
    this.colorScale = d3.scaleSequential(d3.interpolateViridis)

    this.xAxis = d3.axisBottom().scale(this.xScale).tickFormat(this.format)
    this.brush = d3.brushX().extent([
      [margin.left, margin.top],
      [width - margin.right, height - margin.bottom]
    ]).on('brush', this.brushEnd)
  },
  mounted() {
    d3.select(this.$refs.brush).call(this.brush)
  },
  watch: {
    movies: function() {
      this.calculateScales()
      d3.select(this.$refs.xAxis).call(this.xAxis)
    },
    filtered: function() {
      this.calculateData()
    }
  },
  methods: {
    calculateScales: function() {
      if (!this.movies.length) return

      // colors
      const colorDomain = d3.extent(this.movies, d => d.score);
      this.colorScale.domain(colorDomain).nice()

      // x is the attribute
      const xDomain = d3.extent(this.movies, d => d[this.id])
      this.xScale.domain(xDomain).nice()

      // get the bins
      this.histogram.domain(this.xScale.domain())
        .thresholds(this.xScale.ticks(50))
        .value(d => d[this.id])
      const bins = this.histogram(this.movies)

      // calculate y from the bins
      const yMax = d3.max(bins, d => d.length)
      this.yScale.domain([0, yMax])
    },
    calculateData: function() {
      if (!this.filtered.length) return

      // use filtered for bins now that scales are calculated
      const bins = this.histogram(this.filtered)

      // calculate rect bar for each bin
      this.bars = bins.map(d => {
        const {x0, x1} = d
        const x = this.xScale(x0)
        const y = this.yScale(d.length)
        const median = d3.median(d, d => d.score) || 0

        return {
          x,
          width: this.xScale(x1) - x,
          y,
          height: height - margin.bottom - y,
          fill: this.colorScale(median),
        }
      })
    },
    brushEnd: function() {
      let bounds = null;
      if (d3.event.selection) {
        const [x1, x2] = d3.event.selection;
        bounds = [
          this.xScale.invert(x1),
          this.xScale.invert(x2),
        ]
      }
      this.updateFilters({[this.id]: bounds});
    },
  }
}
</script>

<style>
</style>
