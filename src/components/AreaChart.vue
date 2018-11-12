<template>
  <svg class='areachart' :width='width' :height='height'>
    <g class='arcs'>
      <path v-for='d in arcs' :d='d.path' :fill='d.fill' stroke='#fff' />
    </g>
    <g ref='xAxis' :transform='`translate(0, ${y0})`' />
    <g ref='yAxis' :transform='`translate(${margin.left}, 0)`' />
  </svg>
</template>

<script>
import _ from 'lodash'
import * as d3 from 'd3'

const width = 1000
const height = 300
const margin = {top: 0, right: 0, bottom: 20, left: 60}

export default {
  name: 'areachart',
  props: ['movies', 'filtered'],
  data() {
    return {
      width, height, margin,
      arcs: [],
      medianBox: 0,
      y0: 0,
    }
  },
  created() {
    this.xScale = d3.scaleTime().range([margin.left, width - margin.right])
    this.yScale = d3.scaleLinear().range([height - margin.bottom, margin.top])
    this.colorScale = d3.scaleSequential(d3.interpolateViridis)

    this.areaGen = d3.area().curve(d3.curveCatmullRom)

    this.xAxis = d3.axisBottom().scale(this.xScale).tickFormat(this.format)
    this.yAxis = d3.axisLeft().scale(this.yScale)
      .tickSizeOuter(0)
      .tickFormat(d => `${d3.format('$')(parseInt(d / 1000000))}M`)
  },
  watch: {
    movies: function() {
      this.calculateScales()
      d3.select(this.$refs.xAxis).call(this.xAxis)
      d3.select(this.$refs.yAxis).call(this.yAxis)
        .select('.domain').remove()
    },
    filtered: function() {
      this.calculateData()
    }
  },
  methods: {
    calculateScales: function() {
      if (!this.movies.length) return

      // calculate median box office
      this.medianBox = d3.median(this.movies, d => d.boxOffice)

      // colors
      const colorDomain = d3.extent(this.movies, d => d.score);
      this.colorScale.domain(colorDomain).nice()

      // x scale with dates
      const [minDate, maxDate] = d3.extent(this.movies, d => d.date)
      this.xScale.domain([
        d3.timeMonth.offset(minDate, -2),
        d3.timeMonth.offset(maxDate, 2),
      ])

      // y scale with boxOffice - medianBox
      const yExtent = d3.extent(this.movies, d => d.boxOffice - this.medianBox)
      this.yScale.domain(yExtent)
      this.y0 = this.yScale(0)

      // update area generater
      this.areaGen.y0(this.y0)
    },
    calculateData: function() {
      if (!this.filtered.length) return

      // calculate arcs from the filtered movies
      this.arcs = _.chain(this.filtered)
        // put biggest arcs in the background
        .sortBy(d => -Math.abs(d.boxOffice - this.medianBox))
        .map(d => {
          // for each arc, just need d & fill
          return {
            path: this.areaGen([
              [this.xScale(d3.timeMonth.offset(d.date, -2)), this.y0],
              [this.xScale(d.date), this.yScale(d.boxOffice - this.medianBox)],
              [this.xScale(d3.timeMonth.offset(d.date, 2)), this.y0],
            ]),
            fill: this.colorScale(d.score),
            data: d,
          }
        }).value()
    }
  }
}
</script>

<style>
</style>
