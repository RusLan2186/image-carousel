<script setup lang="ts">
import { ref, onMounted } from "vue";
import ImageCarousel from "./components/ImageCarousel.vue";

const images = ref([]);

onMounted(async () => {
  const response = await fetch("https://picsum.photos/v2/list?limit=20");
  const data = await response.json();
  images.value = data;
});
</script>

<template>
  <main>
    <h1 class="title">Image Carousel</h1>
    <ImageCarousel :images="images" />
  </main>
</template>

<style scoped>
.title {
  text-align: center;
  margin-bottom: 14px;

}
</style>
