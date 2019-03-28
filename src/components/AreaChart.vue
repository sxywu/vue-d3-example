<template>
  <div class='areachart'>
    <svg :width='width' :height='height'>
      <!-- MOVIE ARCS -->
      <!-- <transition-group tag='g' :css='false' @enter='enter' @leave='leave'> -->
      <!-- </transition-group> -->
      <!-- AXES -->
      <g ref='xAxis' :transform='`translate(0, ${y0})`' />
      <g ref='yAxis' :transform='`translate(${margin.left}, 0)`' />
    </svg>
  </div>
</template>

<script>
import _ from 'lodash'
import * as d3 from 'd3'
import {TweenLite} from 'gsap'

const width = 1000
const height = 300
const margin = {top: 0, right: 0, bottom: 20, left: 60}

export default {
  name: 'areachart',
  props: ['movies', 'filtered', 'holidays'],
  data() {
    return {
      width, height, margin,
      arcs: [],
      texts: [],
      medianBox: 0,
      y0: 0,
      hovered: null,
    }
  },
  watch: {
    movies: function() {
      this.calculateScales()
      this.calculateData()
    },
  },
  methods: {
    calculateScales: function() {
      if (!this.movies.length) return

      // calculate median box office

      // colors with metascores

      // x scale with dates

      // y scale with boxOffice - medianBox

      // this.y0 = this.yScale(0)

      // update area generater
    },
    calculateData: function() {
      if (!this.filtered.length) return

      // calculate arcs from the filtered movies
    },
    renderAxes: function() {
      const xAxis = d3.axisBottom().scale(this.xScale).tickFormat(this.format)
      const yAxis = d3.axisLeft().scale(this.yScale)
        .tickSizeOuter(0)
        .tickFormat(d => `${d3.format('$')(parseInt(d / 1000000))}M`)

      d3.select(this.$refs.xAxis).call(xAxis)
      d3.select(this.$refs.yAxis).call(yAxis)
        .select('.domain').remove()
    },
    enter: function (el, done) {
      TweenLite.fromTo(el, 0.25,
        {scaleY: 0, svgOrigin: `0 ${this.y0}`},
        {scaleY: 1, onComplete: done}
      )
    },
    leave: function (el, done) {
      TweenLite.fromTo(el, 0.25,
        {scaleY: 1},
        {scaleY: 0, svgOrigin: `0 ${this.y0}`, onComplete: done}
      )
    },
  }
}
</script>

<style>
.areachart {
  position: relative;
}

.arc {
  cursor: pointer;
}

.hovered {
  position: absolute;
  padding: 5px 10px;
  background-color: #fff;
  box-shadow: 0 0 5px #cfcfcf;
  transform: translate(-50%);
  pointer-events: none;
  line-height: 1.6;
  width: 240px;
}
</style>
