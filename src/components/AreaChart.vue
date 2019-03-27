<template>
  <div class='areachart'>
    <svg :width='width' :height='height'>
      <!-- HOLIDAYS -->
      <g class='holidays'>
        <g v-for='d in texts' :transform='`translate(${d.x}, ${36})`'>
          <line :x1='-d.width / 2' :x2='d.width / 2' :stroke='d.fill' />
          <text dy='-.35em' text-anchor='middle' :fill='d.fill'>{{ d.text }}</text>
        </g>
      </g>
      <!-- MOVIE ARCS -->
      <transition-group tag='g' :css='false' @enter='enter' @leave='leave'>
        <path v-for='d in arcs' class='arc' :key='d.id' :d='d.path' :fill='d.fill' stroke='#fff'
          @mouseenter='() => hovered = d' @mouseleave='() => hovered = null' />
      </transition-group>
      <!-- AXES -->
      <g ref='xAxis' :transform='`translate(0, ${y0})`' />
      <g ref='yAxis' :transform='`translate(${margin.left}, 0)`' />
    </svg>
    <div v-if='hovered' class='hovered' :style='{
      top: `${hovered.y + 20}px`,
      left: `${hovered.x}px`,
      }'>
      <strong>title</strong> {{ hovered.data.title }}<br />
      <strong>date</strong> {{ formatDate(hovered.data.date) }}<br />
      <strong>metascore</strong> {{ hovered.data.score }}<br />
      <strong>box office</strong> {{ formatBox(hovered.data.boxOffice) }}<br />
    </div>
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
      this.calculateHolidays()
      this.renderAxes()
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
      this.colorScale = d3.scaleSequential(d3.interpolateViridis).domain(colorDomain).nice()

      // x scale with dates
      const [minDate, maxDate] = d3.extent(this.movies, d => d.date)
      this.xScale = d3.scaleTime().domain([
        d3.timeMonth.offset(minDate, -2),
        d3.timeMonth.offset(maxDate, 2),
      ]).range([margin.left, width - margin.right])

      // y scale with boxOffice - medianBox
      const yExtent = d3.extent(this.movies, d => d.boxOffice - this.medianBox)
      this.yScale = d3.scaleLinear().domain(yExtent).range([height - margin.bottom, margin.top])
      this.y0 = this.yScale(0)

      // update area generater
      this.areaGen = d3.area().curve(d3.curveCatmullRom).y0(this.y0)
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
    calculateData: function() {
      if (!this.filtered.length) return

      // calculate arcs from the filtered movies
      this.arcs = _.chain(this.filtered)
        // put biggest arcs in the background
        .sortBy(d => -Math.abs(d.boxOffice - this.medianBox))
        .map(d => {
          const x = this.xScale(d.date)
          const y = this.yScale(d.boxOffice - this.medianBox)
          // for each arc, just need d & fill
          return {
            id: d.id,
            path: this.areaGen([
              [this.xScale(d3.timeMonth.offset(d.date, -2)), this.y0],
              [x, y],
              [this.xScale(d3.timeMonth.offset(d.date, 2)), this.y0],
            ]),
            fill: this.colorScale(d.score),
            data: d, x, y,
          }
        }).value()
    },
    calculateHolidays: function() {
      this.texts = _.map(this.holidays, (d, i) => {
        const [d1, d2] = d;
        const x1 = this.xScale(d1);
        const x2 = this.xScale(d2);
        return {
          x: (x1 + x2) / 2, width: x2 - x1,
          fill: i % 2 === 0 ? 'rgb(253, 174, 97)' : 'rgb(116, 173, 209)',
          text: i % 2 === 0 ? 's' : 'w',
        };
      });
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
    formatDate: function(date) {
      return d3.timeFormat('%b %d, %Y')(date)
    },
    formatBox: function(box) {
      return d3.format("$,.0f")(box)
    }
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
