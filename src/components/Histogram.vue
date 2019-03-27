<template>
  <svg class='histogram' :width='width' :height='height'>
    <g class='bars'>
      <rect v-for='d in bars' :key='d.id' :x='d.x' :width='d.width'
        :y='d.y' :height='d.height' :fill=d.fill :stroke='d.fill' />
    </g>
    <g ref='xAxis' :transform='`translate(0, ${height - margin.bottom})`' />
    <g ref='brush' />
  </svg>
</template>

<script>
import _ from 'lodash'
import * as d3 from 'd3'
import {TweenMax} from 'gsap'

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
  mounted() {
    this.brush = d3.brushX().extent([
      [margin.left, margin.top],
      [width - margin.right, height - margin.bottom]
    ]).on('brush', this.brushEnd)
    .on('end', this.brushEnd)

    d3.select(this.$refs.brush).call(this.brush)
  },
  watch: {
    movies: function() {
      this.calculateScales()
      this.renderAxes()
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
      this.colorScale = d3.scaleSequential(d3.interpolateViridis)
        .domain(colorDomain).nice()

      // x is the attribute
      const xDomain = d3.extent(this.movies, d => d[this.id])
      this.xScale = d3.scaleLinear().domain(xDomain).nice()
        .range([margin.left, width - margin.right])

      // get the bins
      this.histogram = d3.histogram().domain(this.xScale.domain())
        .thresholds(this.xScale.ticks(50))
        .value(d => d[this.id])
      const bins = this.histogram(this.movies)

      // calculate y from the bins
      const yMax = d3.max(bins, d => d.length)
      this.yScale = d3.scaleLinear().domain([0, yMax])
        .range([height - margin.bottom, margin.top])
    },
    renderAxes: function() {
      const xAxis = d3.axisBottom().scale(this.xScale).tickFormat(this.format)
      d3.select(this.$refs.xAxis).call(xAxis)
    },
    calculateData: function() {
      if (!this.filtered.length) return

      // use filtered for bins now that scales are calculated
      const bins = this.histogram(this.filtered)

      // calculate rect bar for each bin
      this.bars = bins.map((d, i) => {
        const {x0, x1} = d
        const x = this.xScale(x0)
        const y = this.bars[i] ? this.bars[i].toY : this.yScale(0)
        const toY = this.yScale(d.length)
        const fill = this.colorScale(d3.median(d, d => d.score) || 0)

        return {
          id: i,
          x,
          width: this.xScale(x1) - x,
          y, toY,
          height: height - margin.bottom - y,
          toHeight: height - margin.bottom - toY,
          fill,
        }
      })

      // and then animate them
      const tween = TweenMax.staggerFromTo(this.bars, 0.25, {
        // from
        cycle: {
          y: (i) => this.bars[i].y,
          height: (i) => this.bars[i].height,
        }
      }, {
        // to
        cycle: {
          y: (i) => this.bars[i].toY,
          height: (i) => this.bars[i].toHeight,
        },
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

<style scoped>
</style>
