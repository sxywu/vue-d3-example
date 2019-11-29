<template>
  <div class='areachart'>
    <svg :width='width' :height='height'>
      <!-- MOVIE ARCS -->
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
    }
  },
  watch: {
  },
  methods: {
    renderAxes: function() {
      if (!this.xScale || !this.yScale) return

      const xAxis = d3.axisBottom().scale(this.xScale).tickFormat(this.format)
      const yAxis = d3.axisLeft().scale(this.yScale)
        .tickSizeOuter(0)
        .tickFormat(d => `${d3.format('$')(parseInt(d / 1000000))}M`)

      // call axes on group elements
      d3.select(this.$refs.xAxis).call(xAxis)
      d3.select(this.$refs.yAxis).call(yAxis)
        .selectAll('path').remove()
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
</style>
