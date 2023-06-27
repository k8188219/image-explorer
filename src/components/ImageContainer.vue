<script setup lang="ts">
import type { PropType } from "vue";
import { ref, watchEffect, watch, nextTick } from "vue";

const props = defineProps({
  images: {
    type: Array as PropType<URL[]>,
    default: () => [],
  },
  scale: {
    type: Number,
    default: 1,
  },
});

const container = ref<Element>();
const width = ref(0.6);
const image_list = ref<{ src: string; loaded: boolean }[]>([]);
// window.onbeforeunload = function confirmExit() { return " " };

watchEffect(() => {
  image_list.value = props.images.map((src) => {
    var img = { src: `${src}`, loaded: false };
    return img;
  });
});

watch(
  () => props.scale,
  async (scale, prevScale) => {
    if (!container.value) return;
    const documentElement = document.documentElement;
    var y =
      (documentElement.scrollTop + documentElement.clientHeight / 2) /
      prevScale;
    var x =
      (container.value.scrollLeft + container.value.clientWidth / 2) /
      prevScale;
    width.value = 0.6 * scale;
    await nextTick();
    documentElement.scrollTop = y * scale - documentElement.clientHeight / 2;
    container.value.scrollLeft = x * scale - container.value.clientWidth / 2;
  }
);

defineExpose({ el: container });
</script>

<template>
  <div ref="container" class="overflow-auto" :style="{ '--width': width }">
    <div
      v-for="image in image_list"
      :key="image.src"
      class="my-0 mx-auto flex justify-center"
      :style="{
        width: 'calc(100% * var(--width))',
        paddingTop: image.loaded ? '' : 'calc(75% * var(--width))',
      }"
    >
      <img
        :src="image.src"
        class="w-full"
        :style="{ height: image.loaded ? '' : '0px' }"
        @load="image.loaded = true"
        loading="lazy"
      />
    </div>
  </div>
</template>
