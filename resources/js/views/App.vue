<template>
  <v-app>
    <v-card tile>
      <v-toolbar color="#658DB2" dark flat dense>
        <v-app-bar-nav-icon></v-app-bar-nav-icon>

        <v-toolbar-title class="title text-none">
          AgBird
          <span class="subtitle-1 text-none">- NeoLink</span>
        </v-toolbar-title>

        <template v-slot:extension>
          <v-tabs v-model="tab" centered background-color="transparent">
            <v-tabs-slider color="white"></v-tabs-slider>

            <v-tab v-for="item in items" :key="item.id">
              <v-icon v-if="item.id == 1" class="mx-2">mdi-chart-bell-curve-cumulative</v-icon>
              <v-icon v-else-if="item.id == 2" class="mx-2">mdi-airplane</v-icon>
              {{ item.name }}
            </v-tab>
          </v-tabs>
        </template>
      </v-toolbar>

      <v-tabs-items v-model="tab">
        <v-tab-item v-for="item in items" :key="item.id">
          <v-card flat>
            <v-card-text class="my-3">{{item.text}}</v-card-text>
            <signal-section v-if="item.id == 1"></signal-section>
            <images-section v-if="item.id == 2"></images-section>
          </v-card>
        </v-tab-item>
      </v-tabs-items>
    </v-card>
  </v-app>
</template>

<script>
import SignalSection from '@/js/components/SignalSection';
import ImagesSection from '@/js/components/ImagesSection';

export default {
  components: {
    SignalSection,
    ImagesSection
  },
  data() {
    return {
      // graphData: [],
      idleTimeout: 0,
      tab: null,
      items: [
        {
          id: 1,
          name: 'Señales',
          text: 'Resumen señal registrada'
        },
        {
          id: 2,
          name: 'Imágenes',
          text: 'Imagen registrada'
        }
      ]
    };
  },
  methods: {
    testServer: function() {
      axios('http://localhost:80', {
        method: 'GET',
        mode: 'no-cors',
        headers: {
          'Access-Control-Allow-Origin': '*',
          'Content-Type': 'text/html; charset=utf-8'
        },
        withCredentials: false,
        credentials: 'same-origin'
      }).then(response => console.log(response));
    }
  }
};
</script>

<style>
</style>