<script setup lang="ts">
import WebFrame from "@/components/WebFrame.vue";
import ImageContainer from "@/components/ImageContainer.vue";
import { ref, watch } from "vue";

const file_manager_open = ref(true);

const container = ref<InstanceType<typeof ImageContainer>>();
const scale = ref(1);
const images = ref<URL[]>([]);
// window.onbeforeunload = function confirmExit() { return " " };

function getBaseLog(x: number, y: number) {
  return Math.log(y) / Math.log(x);
}

function triggerEvent(delta: number) {
  var a = getBaseLog(1.2, scale.value);
  scale.value = 1.2 ** (a + delta / 100);

  if (scale.value < 10 / 60) scale.value = 10 / 60;
}
function wheel_event(e: Event) {
  if (!(e instanceof WheelEvent)) return;
  triggerEvent(-e.deltaY);
}

// bind touch events to ImageContainer
watch(
  () => container.value?.el,
  (container, pervContainer, onCleanup) => {
    if (!container) return;

    var dist = null as number | null;
    const listen_touch = (e: Event) => {
      if (!(e instanceof TouchEvent)) return;
      const { touches } = e;
      if (touches.length === 2) {
        e.preventDefault();
      }
    };
    const touch_end = () => {
      dist = null;
    };
    const touch_move = (e: Event) => {
      if (!(e instanceof TouchEvent)) return;
      if (!(e.touches.length > 2)) return;
      var prev = dist;
      dist = Math.hypot(
        e.touches[0].pageX - e.touches[1].pageX,
        e.touches[0].pageY - e.touches[1].pageY
      );
      const movement = prev === null ? 0 : dist - prev;
      triggerEvent(movement);
    };
    container.addEventListener("touchstart", listen_touch, { passive: false });
    window.addEventListener("touchend", touch_end);
    window.addEventListener("touchmove", touch_move);

    onCleanup(() => {
      container.removeEventListener("touchstart", listen_touch);
      window.removeEventListener("touchend", touch_end);
      window.removeEventListener("touchmove", touch_move);
    });
  }
);
</script>

<template>
  <div class="flex">
    <div
      class="fixed inset-y-0 overflow-hidden bg-white transition-[width] duration-300 ease-in-out"
      :class="{ 'w-0': !file_manager_open, 'w-2/5': file_manager_open }"
    >
      <WebFrame @insert_images="images = $event" />
    </div>
    <ImageContainer
      ref="container"
      class="ml-auto"
      :class="{ 'w-full': !file_manager_open, 'w-3/5': file_manager_open }"
      :images="images"
      :scale="scale"
      @click="file_manager_open = !file_manager_open"
      @wheel.ctrl.prevent="wheel_event"
    />
  </div>
</template>
