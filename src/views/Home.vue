<template>
  <div class="home">
    <AudioAnalysis></AudioAnalysis>
  </div>
</template>

<script>
// @ is an alias to /src
import AsyncLoading from "@/components/AnalysisLoading.vue";
import AsyncLoadError from "@/components/AnalysisLoadError.vue";

export default {
  name: "Home",
  components: {
    AudioAnalysis: () => ({
      component: import("@/components/AudioAnalysis.vue"),
      loading: AsyncLoading,
      error: AsyncLoadError,
      delay: 100,
      timeout: 100000,
    }),
  },
};
</script>
