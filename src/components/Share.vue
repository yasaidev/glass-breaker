<template>
  <v-btn class="white--text" @click="Tweet" color="blue">
    Tweet
    <v-icon color="white" small>fab fa-twitter</v-icon>
  </v-btn>
</template>

<script>
export default {
  name: "ShareButton",
  props: {
    Freq: {
      type: Number,
    },
  },
  methods: {
    Tweet: function () {
      if (typeof this.Freq !== "undefined" && this.Freq !== 0) {
        location.href =
          "https://twitter.com/intent/tweet?text=" +
          this.Freq +
          "Hzで破壊できそう😎%0a" +
          "%20%23ガラスブレイカー" +
          "&url" +
          location.href;
      } else {
        location.href =
          "https://twitter.com/intent/tweet?text=" +
          "ガラス破壊してぇ🥺%0a" +
          "%20%23ガラスブレイカー" +
          "&url" +
          location.href;
      }
    },
  },
};
</script>

<style>
</style>