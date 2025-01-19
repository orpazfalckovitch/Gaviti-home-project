<script lang="ts">
import { IMovie } from '@/models/movie.interface'
import { useRoute } from 'vue-router'
export default {
  data() {
    return { movie: {} as IMovie }
  },
  methods: {
    //call popular movies API
    fetchData() {
      const route = useRoute()
      const movieId = route.params.id

      const options = {
        method: 'GET',
        headers: {
          accept: 'application/json',
          Authorization:
            'Bearer eyJhbGciOiJIUzI1NiJ9.eyJhdWQiOiJjNzhkZmFhNjJhMTU1MmM2OWY5MGVkMWZjYjdiY2E3OSIsIm5iZiI6MTczNzI2OTU3Ny44NjQsInN1YiI6IjY3OGNhMTQ5NjhlMGQ4NzM2MzZkZjBkMCIsInNjb3BlcyI6WyJhcGlfcmVhZCJdLCJ2ZXJzaW9uIjoxfQ.AjmACrhlZmtlrWraxjiamm2QlxO3ngGSDaYsYkIaCrM',
        },
      }

      fetch(`https://api.themoviedb.org/3/movie/${movieId}`, options)
        .then((res) => res.json())
        .then((res) => {
          this.movie = res
          console.log(res)
        })
        .catch((err) => console.error(err))
    },

    backButton() {
      this.$router.go(-1)
    },
  },
  created() {
    this.fetchData()
  },
}
</script>

<template>
  <div class="backButton" @click="backButton()"><</div>
  <div class="movie-view">
    <img
      class="poster"
      :src="'https://image.tmdb.org/t/p/w500' + movie.poster_path"
      :alt="movie.title"
    />
    <div class="movie-details">
      <h3 class="movie-title">{{ movie.title }}</h3>
      <p>{{ movie.overview }}</p>
      <p>{{ movie.release_date }}</p>
    </div>
  </div>
</template>

<style scoped lang="scss">
.backButton {
  color: red;
  font-size: 20px;
  cursor: pointer;
}
.movie-view {
  display: flex;
  flex-direction: row;
  .poster {
    width: 20%;
    height: auto;
  }
  .movie-details {
    .movie-title {
      color: red;
      font-size: larger;
      font-weight: bold;
    }
    margin-left: 20px;
  }
}
</style>
