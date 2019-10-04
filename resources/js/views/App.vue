<template>
  <v-app>
    <v-card>
      <v-toolbar color="#4D6B87" dark flat>
        <v-app-bar-nav-icon></v-app-bar-nav-icon>

        <v-toolbar-title>Data Logger</v-toolbar-title>

        <template v-slot:extension>
          <v-tabs v-model="tab" centered background-color="transparent">
            <v-tabs-slider color="white"></v-tabs-slider>

            <v-tab v-for="item in items" :key="item.id">
              <v-icon v-if="item.id == 1" class="mx-2">mdi-database</v-icon>
              <v-icon v-else-if="item.id == 2" class="mx-2">mdi-chart-bell-curve-cumulative</v-icon>
              {{ item.name }}
            </v-tab>
          </v-tabs>
        </template>
      </v-toolbar>

      <v-tabs-items v-model="tab">
        <v-tab-item v-for="item in items" :key="item.id">
          <v-card flat>
            <v-card-text v-text="text"></v-card-text>
          </v-card>
        </v-tab-item>
      </v-tabs-items>
    </v-card>
  </v-app>
</template>

<script>
export default {
  data() {
    return {
      tab: null,
      items: [
        {
          id: 1,
          name: 'Señales'
        },
        {
          id: 2,
          name: 'Imágenes'
        }
      ],
      text:
        'Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat.'
    };
  }
};
</script>

<style>
</style>