<template>
  <v-btn depressed class="white--text" @click="Tweet" color="blue">
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
        this.$gtag.event("クリック計測", {
          event_category: "SNSボタンクリック",
          event_label: "twitter(計測結果)",
          value: 1,
        });
        location.href =
          "https://twitter.com/intent/tweet?text=" +
          "私のグラスは" +
          this.Freq +
          "Hzで破壊できそう😎%0a" +
          "%20%23爆音グラス破壊" +
          "&url=" +
          location.href;
      } else {
        this.$gtag.event("クリック計測", {
          event_category: "SNSボタンクリック",
          event_label: "twitter(紹介)",
          value: 1,
        });
        location.href =
          "https://twitter.com/intent/tweet?text=" +
          "ガラス破壊してぇ🥺%0a" +
          "%20%23爆音グラス破壊" +
          "&url=" +
          location.href;
      }
    },
  },
};
</script>

<style>
</style>