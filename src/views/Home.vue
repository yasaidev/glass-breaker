<template>
  <div class="home">
    <AudioAnalysis></AudioAnalysis>
  </div>
</template>

<script>
// @ is an alias to /src
import AudioAnalysis from "@/components/AudioAnalysis.vue";

export default {
  name: "Home",
  components: {
    AudioAnalysis,
  },
};
</script>
