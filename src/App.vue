<template>
  <div id="app">
    <!-- HEADER -->
    <h1>The Seasonality of Box Office Hits</h1>
    <p class='description'>
In the last decade, most block buster hits have happened around or during the summer and winter holidays.
<strong>Brush</strong> the histograms to filter the movies by metascore and/or box office figures, and <strong>hover</strong> the movies to see more details.
    </p>
    <!-- AREA CHART -->
    <div class='area-container'>
      <div class='area-title'>
        <strong>$ over/under median box office</strong>
      </div>
      <AreaChart v-bind='{movies, filtered}' />
    </div>
    <!-- HISTOGRAM -->
    <div class='histograms' v-for='({id, label, format}) in histograms'>
      <Histogram v-bind='{movies, filtered, id, format, updateFilters}' />
      <div>
        <strong>{{ label }}</strong>
      </div>
    </div>

  </div>
</template>

<script>
import _ from 'lodash'
import * as d3 from 'd3'
import AreaChart from './components/AreaChart'
import Histogram from './components/Histogram'

const startYear = 2008;

export default {
  name: 'app',
  components: {
    Histogram, AreaChart,
  },
  data() {
    return {
      histograms: [
        {id: 'score', label: 'metascores'},
        {id: 'boxOffice', label: 'box office figures', format: d => `$${parseInt(d/ 1000000)}M`},
      ],
      movies: [],
      filters: {},
      filtered: [],
    }
  },
  mounted() {
    fetch(`./movies.json`)
      .then(resp => resp.json())
      .then(movies => {
        this.movies = _.chain(movies)
          .map(d => Object.assign(d, {date: new Date(d.date)}))
          .filter(d => d.boxOffice && d.year >= startYear)
          .value();
        this.filtered = this.movies
      });
  },
  // computed: {
  //   filtered: function() {
  //     return _.filter(this.movies, d => _.every(this.filters, (bounds, key) =>
  //       !bounds || bounds[0] < d[key] && d[key] < bounds[1]))
  //   }
  // },
  methods: {
    updateFilters: function(filter) {
      this.filters = Object.assign(this.filters, filter)
      this.filtered = _.filter(this.movies, d =>
        _.every(this.filters, (bounds, key) => !bounds || bounds[0] < d[key] && d[key] < bounds[1]))
    }
  }
}
</script>

<style>
#app {
  width: 1000px;
  margin: auto;
  text-align: center;
}

.description {
  width: 600px;
  line-height: 2;
  margin: 20px auto;
}

.area-container {
  position: relative;
}

.area-title {
  position: absolute;
  left: 65px;
}

.histograms {
  display: inline-block;
}
</style>
