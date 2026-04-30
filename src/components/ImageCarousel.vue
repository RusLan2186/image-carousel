<script setup>
import { ref, computed, onMounted, onUnmounted } from "vue";

const props = defineProps({
  images: {
    type: Array,
    required: true,
  },
});

const ITEM_WIDTH = 212;

const visibleCount = ref(1);
const currentIndex = ref(0);

const isMobile = ref(false);
const viewportRef = ref(null);

const SWIPE_THRESHOLD = 50;
const DRAG_CLICK_THRESHOLD = 8;

const isPointerDown = ref(false);
const startX = ref(0);
const currentX = ref(0);
const didDrag = ref(false);
const suppressClick = ref(false);

const updateVisibleCount = () => {
  isMobile.value = window.innerWidth <= 768
  const viewport = document.querySelector(".carousel__viewport");
  if (viewport) {
    visibleCount.value = isMobile.value
      ? 1
      : Math.max(1, Math.floor(viewport.offsetWidth / ITEM_WIDTH));
  }
};

onMounted(() => {
  updateVisibleCount();
  window.addEventListener("resize", updateVisibleCount);
});

onUnmounted(() => {
  window.removeEventListener("resize", updateVisibleCount);
});


const next = () => {
  if (!props.images.length) return;
  currentIndex.value = (currentIndex.value + 1) % props.images.length;
};

const prev = () => {
  if (!props.images.length) return;
  currentIndex.value =
    (currentIndex.value - 1 + props.images.length) % props.images.length;
};

const onPointerDown = (event) => {
  if (event.pointerType === "mouse" && event.button !== 0) return;

  isPointerDown.value = true;
  startX.value = event.clientX;
  currentX.value = event.clientX;
  didDrag.value = false;

  if (viewportRef.value) {
    viewportRef.value.setPointerCapture?.(event.pointerId);
  }
};

const onPointerMove = (event) => {
  if (!isPointerDown.value) return;

  currentX.value = event.clientX;
  if (Math.abs(currentX.value - startX.value) > DRAG_CLICK_THRESHOLD) {
    didDrag.value = true;
  }
};

const onPointerEnd = () => {
  if (!isPointerDown.value) return;

  const deltaX = currentX.value - startX.value;

  if (Math.abs(deltaX) >= SWIPE_THRESHOLD) {
    if (deltaX < 0) {
      next();
    } else {
      prev();
    }
  }

  if (didDrag.value) {
    suppressClick.value = true;
    window.setTimeout(() => {
      suppressClick.value = false;
    }, 0);
  }

  isPointerDown.value = false;
};


const visibleImages = computed(() => {
  if (!props.images.length) return [];

  const result = [];
  for (let i = 0; i < visibleCount.value; i++) {
    const index = (currentIndex.value + i) % props.images.length;
    result.push(props.images[index]);
  }
  return result;
});


const selectedImages = ref(new Set());

const toggleSelect = (image) => {
  if (suppressClick.value) return;

  if (selectedImages.value.has(image.download_url)) {
    selectedImages.value.delete(image.download_url);
  } else {
    selectedImages.value.add(image.download_url);
  }
  selectedImages.value = new Set(selectedImages.value);
};

const isSelected = (image) => {
  return selectedImages.value.has(image.download_url);
};
</script>

<template>
  <div class="carousel-wrapper">
    <div class="carousel">
      <button
        class="carousel__btn carousel__btn--prev"
        @click="prev"
      >
       <span>‹</span> 
      </button>

      <div
        ref="viewportRef"
        class="carousel__viewport"
        :class="{ 'carousel__viewport--dragging': isPointerDown }"
        @pointerdown="onPointerDown"
        @pointermove="onPointerMove"
        @pointerup="onPointerEnd"
        @pointercancel="onPointerEnd"
        @pointerleave="onPointerEnd"
      >
        <TransitionGroup name="slide" tag="div" class="carousel__track">
          <div
            v-for="image in visibleImages"
            :key="image.id"
            class="carousel__item"
            :class="{ 'carousel__item--selected': isSelected(image) }"
            @click="toggleSelect(image)"
          >
            <img
              :src="`https://picsum.photos/id/${image.id}/400/300`"
              :alt="image.author"
            />
          </div>
        </TransitionGroup>
      </div>

      <button
        class="carousel__btn carousel__btn--next"
        @click="next"
      >
        <span>›</span>
      </button>
    </div>

    <div class="selected-list" v-if="selectedImages.size > 0">
      <h3>Selected images:</h3>
      <ul>
        <li v-for="url in selectedImages" :key="url">
          <a :href="url" target="_blank" rel="noopener noreferrer">{{ url }}</a>
        </li>
      </ul>
    </div>
  </div>
</template>

<style scoped>
.carousel-wrapper {
  width: 100%;
  padding: 32px 10px;
  box-sizing: border-box;
}

.carousel {
  display: flex;
  align-items: center;
  gap: 16px;
  width: 100%;
}

.carousel__viewport {
  flex: 1;
  overflow: hidden;
  min-width: 0;
  touch-action: pan-y;
}

.carousel__viewport--dragging {
  cursor: grabbing;
}

.carousel__track {
  display: flex;
  justify-content: center;
  gap: 12px;
  padding: 10px 0;
}

.carousel__item {
  flex: 0 0 196px;
  cursor: pointer;
  border-radius: 12px;
  overflow: hidden;
  border: 3px solid transparent;
  transition:
    transform 0.2s,
    border-color 0.2s;
}

.carousel__item:hover {
  transform: translateY(-4px);
}

.carousel__item--selected {
  border-color: #646cff;
}

.carousel__item img {
  width: 100%;
  height: 150px;
  object-fit: cover;
  display: block;
}

.carousel__btn {
  background: rgba(100, 108, 255, 0.15);
  color: #646cff;
  border: 2px solid #646cff;
  border-radius: 50%;
  width: 44px;
  height: 44px;
  font-size: 24px;
  cursor: pointer;
  flex-shrink: 0;
  display: flex;
  align-items: center;
  justify-content: center;
  line-height: 1.5;
  padding: 0;
  transition:
    background 0.2s,
    transform 0.2s;
}

.carousel__btn span{
  position: relative;
  top: -2px;

}

.carousel__btn:hover {
  background: #646cff;
  color: white;
  transform: scale(1.1);
}

.carousel__btn:active {
  transform: scale(0.95);
}


.selected-list {
  margin-top: 10px;
  padding: 10px;
  background: rgba(255, 255, 255, 0.05);
  border-radius: 12px;
  border: 1px solid rgba(255, 255, 255, 0.1);
}

.selected-list h3 {
  margin: 0 0 12px 0;
  color: #ffffff;
  font-size: 14px;
  text-transform: uppercase;
  letter-spacing: 1px;
}

.selected-list ul {
  list-style: none;
  padding: 0;
  margin: 0;
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.selected-list li {
  padding: 8px 12px;
  background: rgba(100, 108, 255, 0.1);
  border-radius: 6px;
  border-left: 3px solid #646cff;
}

.selected-list a {
  font-size: 13px;
  color: #646cff;
  text-decoration: none;
  word-break: break-all;
}

.selected-list a:hover {
  text-decoration: underline;
}

.slide-enter-active,
.slide-leave-active {
  transition: all 0.3s ease;
}

.slide-enter-from {
  opacity: 0;
  transform: translateX(30px);
}

.slide-leave-to {
  opacity: 0;
  transform: translateX(-30px);
}

.slide-leave-active {
  position: absolute;
}


@media (max-width: 768px) {
  .carousel__item {
    flex: 0 0 100%;
  }
}
</style>
