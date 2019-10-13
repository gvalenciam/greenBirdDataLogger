<template>
  <span style="padding: 16px">
    <v-btn
      rounded
      depressed
      class="text-none white--text"
      color="#4D6B87"
      v-on:click="forceRender"
    >Refrescar</v-btn>
    <v-container>
      <v-layout justify-center align-center>
        <v-img
          contain
          :key="componentKey"
          :src="'img/baseImage.png' + '?dummy=' + Math.random()"
          id="imageSlot"
        ></v-img>
      </v-layout>
    </v-container>
  </span>
</template>

<script>
export default {
  name: 'ImagesSection',
  data() {
    return {
      componentKey: 1
    };
  },
  methods: {
    refreshImage: function() {
      $('#imageContainer').empty();
      var node = document.createElement('img');
      node.src = 'img/baseImage.png';
      node.id = 'imageSlot';
      document.getElementById('imageContainer').appendChild(node);
      //   $('#imageContainer').appendChild(
      //     '<v-img contain src="img/baseImage.png" id="imageSlot"></v-img>'
      //   );
      //   $('#imageSlot').attr('src', 'img/baseImage.png?' + d.getTime());
      // $('#imageSlot').src = 'img/baseImage.png';
    },
    forceRender() {
      this.componentKey += 1;
    }
  }
};
</script>

<style scoped>
</style>