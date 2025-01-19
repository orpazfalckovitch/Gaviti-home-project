<script lang="ts">
import { IMovie } from '@/models/movie.interface'
import { Chart, registerables } from 'chart.js'
Chart.register(...registerables)

export default {
  data() {
    return {
      popularMovies: [] as IMovie[],
      loading: true,
      error: null as string | null,
      latestMovies: [] as IMovie[],
      chartData: {
        labels: [] as string[],
        datasets: [{ label: 'Number of Movies', backgroundColor: '#42A5F5', data: [] as number[] }],
      },
      searchMovie: '',
      id: 'chart',
    }
  },
  methods: {
    getOptions() {
      return {
        method: 'GET',
        headers: {
          accept: 'application/json',
          Authorization:
            'Bearer eyJhbGciOiJIUzI1NiJ9.eyJhdWQiOiJjNzhkZmFhNjJhMTU1MmM2OWY5MGVkMWZjYjdiY2E3OSIsIm5iZiI6MTczNzI2OTU3Ny44NjQsInN1YiI6IjY3OGNhMTQ5NjhlMGQ4NzM2MzZkZjBkMCIsInNjb3BlcyI6WyJhcGlfcmVhZCJdLCJ2ZXJzaW9uIjoxfQ.AjmACrhlZmtlrWraxjiamm2QlxO3ngGSDaYsYkIaCrM',
        },
      }
    },

    getSearchData() {
      if (this.searchMovie.length < 3) {
        return
      }
      fetch(
        `https://api.themoviedb.org/3/search/movie?query=${this.searchMovie}`,
        this.getOptions(),
      )
        .then((res) => res.json())
        .then((data) => {
          this.popularMovies = data.results
          this.loading = false
        })
        .catch((err) => {
          console.error(err)
          this.error = 'Error fetching data'
          this.loading = false
        })
    },

    getPopularMovies(data) {
      if (data && data.results) {
        this.popularMovies = data.results
        //Remove duplicate movies by id
        this.popularMovies = this.popularMovies.filter(
          (item, index, array) => array.findIndex((t) => t.id === item.id) === index,
        )
        //Show the release date in a readable format
        //Parse the vote_average to a readable format
        this.popularMovies = this.popularMovies.map((item) => {
          return {
            ...item,
            release_date: new Date(item.release_date).toLocaleDateString(),
            vote_average: Math.round(item.vote_average * 10),
          }
        })
        //Sort movies by release date
        this.popularMovies = this.popularMovies.sort((a, b) => {
          return new Date(b.release_date).getTime() - new Date(a.release_date).getTime()
        })

        this.loading = false
      }
    },

    //call popular movies API
    getPoplarMovies() {
      fetch('https://api.themoviedb.org/3/movie/popular?language=en-US&page=1', this.getOptions())
        .then((res) => res.json())
        .then((data) => {
          this.getPopularMovies(data)
        })
        .catch((err) => {
          console.error(err)
          this.error = 'Error fetching data'
          this.loading = false
        })
    },

    //call latest movies API
    async getLatestMovies() {
      const today = new Date()
      const ninetyDaysAgo = new Date(today.setDate(today.getDate() - 90))
        .toISOString()
        .split('T')[0]

      fetch(
        `https://api.themoviedb.org/3/discover/movie?primary_release_date.gte=${ninetyDaysAgo}`,
        this.getOptions(),
      )
        .then((response) => response.json())
        .then((data) => {
          this.latestMovies = data.results
          this.prepareChartData()
          this.renderChart()
          this.loading = false
        })
        .catch((error) => console.error('Error fetching data:', error))
    },

    prepareChartData() {
      const releaseDates = this.latestMovies.map((movie) => new Date(movie.release_date))
      const dateCounts = releaseDates.reduce((acc: { [key: string]: number }, date) => {
        const dateString = date.toISOString().split('T')[0]
        acc[dateString] = (acc[dateString] || 0) + 1
        return acc
      }, {})

      this.chartData.labels = Object.keys(dateCounts).sort()
      this.chartData.datasets[0].data = this.chartData.labels.map((label) => dateCounts[label])
    },

    renderChart() {
      const canvas = document.getElementById(this.id) as HTMLCanvasElement | null
      if (canvas) {
        const ctx = canvas.getContext('2d')
        if (ctx) {
          new Chart(ctx, {
            type: 'line',
            data: this.chartData,
            options: {
              scales: {
                x: {
                  type: 'time',
                  time: {
                    unit: 'day',
                  },
                },
              },
            },
          })
        }
      }
    },

    goToMovie(id: number) {
      this.$router.push({ name: 'movie', params: { id } })
    },
  },
  created() {
    this.getPoplarMovies()
    this.getLatestMovies()
  },
  mounted() {},
}
</script>

<template>
  <main>
    <div>
      <h1>Popular movies:</h1>
      <div v-if="loading">Loading...</div>
      <div v-if="error">{{ error }}</div>

      <div class="search">
        <input
          type="text"
          placeholder="Search for a movie"
          v-model="searchMovie"
          @input="getSearchData"
        />
      </div>
      <div class="cards" v-if="popularMovies.length">
        <div class="card" v-for="item in popularMovies" :key="item.id" @click="goToMovie(item.id)">
          <img
            class="poster"
            :src="'https://image.tmdb.org/t/p/w500' + item.poster_path"
            :alt="item.title"
          />
          <div>
            <div>
              <span>{{ item.vote_average }}</span>
              <span>⭐</span>
            </div>
            <h2>{{ item.title }}</h2>
            <p>{{ item.release_date }}</p>
          </div>
        </div>
      </div>

      <div class="chart">
        <h1>Movies Released in the Last 90 Days</h1>
        <line-chart :chart-data="chartData"></line-chart>
      </div>

      <canvas :id="id"></canvas>
    </div>
  </main>
</template>

<style scoped lang="scss">
.search {
  margin-bottom: 28px;
  input {
    width: 500px;
    height: 50px;
    font-size: 25px;
  }
}
.cards {
  display: flex;
  flex-direction: row;
  justify-content: space-around;
  .card {
    margin: 0 20px;
    .poster {
      height: 200px;
      width: 200px;
    }
  }
}
</style>
