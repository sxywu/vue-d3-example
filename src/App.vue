<template>
  <div id="app">
    <h1>The Seasonality of Box Office Hits</h1>
    <p class='description'>
In the last decade, most block buster hits have happened around or during the summer and winter holidays.  <strong>Brush</strong> the histograms to filter the movies by metascore and/or box office figures, and <strong>hover</strong> the movies to see more details.
    </p>
  </div>
</template>

<script>
import _ from 'lodash'
import * as d3 from 'd3'

const startYear = 2008;
const numYears = 10;

export default {
  name: 'app',
  components: {
  },
  data() {
    return {
      movies: [],
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
      });
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
</style>
