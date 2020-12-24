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
    // AudioAnalysis: () => import("@/components/AudioAnalysis.vue"),
    AudioAnalysis: () => ({
      // The component we want to load.
      component: import("@/components/AudioAnalysis.vue"),
      // The component to use as a placeholder while the
      // async component is loading.
      loading: AsyncLoading,
      // The component to render instead if there is an error
      // loading the async component.
      error: AsyncLoadError,
      // The delay before the loading component is shown.
      delay: 100,
      // If this timeout is reached, the async component is considered
      // to have failed loading.
      timeout: 100000,
    }),
  },
};
</script>
