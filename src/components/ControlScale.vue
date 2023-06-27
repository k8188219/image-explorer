<script setup lang="ts">
import { ref, watch } from "vue";

const container = ref();
const scale = ref(1);
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
      if (!(e.touches.length >= 2)) return;
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
  <slot v-bind="{ scale, wheel_event, ref: container }"></slot>
</template>
