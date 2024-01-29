<template>
  <ion-page>
    <ion-header :translucent="true">
      <ion-toolbar>
        <ion-buttons slot="start">
          <ion-menu-button color="primary"></ion-menu-button>
        </ion-buttons>
        <ion-title>{{ $route.params.id }}</ion-title>
      </ion-toolbar>
    </ion-header>

    <!-- Enable scroll events and create a ref -->
    <ion-content :fullscreen="true" scrollEvents ref="content">
      <ion-refresher slot="fixed" @ionRefresh="doRefresh($event)">
        <ion-refresher-content></ion-refresher-content>
      </ion-refresher>
      <ion-header collapse="condense">
        <ion-toolbar>
          <ion-title size="large">{{ $route.params.id }}</ion-title>
        </ion-toolbar>
      </ion-header>
    
      <div id="container">
        <div v-for="movie in movies" :key="movie.id">
          <MovieCard v-bind:movie="movie"></MovieCard>
        </div>
      </div>
      <ion-infinite-scroll
          @ionInfinite="loadData($event)"
          threshold="100px"
          id="infinite-scroll"
          :disabled="isDisabled">
          <ion-infinite-scroll-content
              loading-spinner="bubbles"
              loading-text="Loading more movies...">
          </ion-infinite-scroll-content>
      </ion-infinite-scroll>
    </ion-content>
  </ion-page>
</template>

<script>
import { IonButtons, IonContent, IonHeader, IonMenuButton, IonPage, IonTitle, IonToolbar, 
IonInfiniteScroll, IonInfiniteScrollContent, IonRefresher, IonRefresherContent, } from '@ionic/vue';

import { ref } from "vue";
import MovieCard from "../MovieCard.vue";
import axios from "axios";

export default {
  name: 'Folder',
  components: {
    IonButtons,
    IonContent,
    IonHeader,
    IonMenuButton,
    IonPage,
    IonTitle,
    IonToolbar,
    MovieCard,
    IonInfiniteScroll,
    IonInfiniteScrollContent,
    IonRefresher,
    IonRefresherContent,
  },
  data() {
    return {
      movies: ref([]),
      pageNumber: 1,
      maxPages: 1,
      endpoint: this.$route.params.id
                .toLowerCase()
                .split(" ")
                .join("_"),
      country: "",
    };
  },
  methods: {
    async fetch(pageNumber) {
      const movies = await axios.get(
          "https://api.themoviedb.org/3/movie/" +
              this.endpoint +
              "?api_key=395323a2055d57bacc9cdc4bc1ea14ed&language=en-US&page=" +
              pageNumber +
              "&region=" +
              this.country
      );
      this.movies = movies.data.results;
      this.pageNumber = movies.data.page + 1;
      this.maxPages = movies.data.total_pages;
    },
    async pushData(pageNumber) {
      const movies = await axios.get(
          "https://api.themoviedb.org/3/movie/" +
              this.endpoint +
              "?api_key=395323a2055d57bacc9cdc4bc1ea14ed&language=en-US&page=" +
              pageNumber +
              "&region=" +
              this.country
      );
      this.movies = this.movies.concat(movies.data.results);
      this.pageNumber = movies.data.page + 1;
      this.maxPages = movies.data.total_pages;
    },
    async loadData(ev) {
      const res = await this.pushData(this.pageNumber);
      console.log("Loaded data");
      console.log(res);
      console.log(this.pageNumber);
      ev.target.complete();
      if (this.pageNumber >= this.maxPages) {
          ev.target.disabled = true;
      }
    },
    async doRefresh(event) {
      const res = await this.fetch(1);
      console.log(res);
      event.target.complete();
    },
  },
  mounted() {
    this.fetch(this.pageNumber);
  },
  watch: {
    $route(to, from) {
      this.endpoint = this.$route.params.id
          .toLowerCase()
          .split(" ")
          .join("_");
      this.pageNumber = 1;
      this.maxPages = 1;

      this.fetch(this.pageNumber);
      // Scroll to top when the tab changes
      this.scrollToTop();
    }
  },
    setup() {
      // Get ref to content
      const content = ref();

      // Add function to scroll to top
      return {
        content,
        scrollToTop: () => content.value.$el.scrollToTop(),
      };
    },
}
</script>

<style scoped>
</style>