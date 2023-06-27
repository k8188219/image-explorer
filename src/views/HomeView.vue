<script setup lang="ts">
import { ref } from "vue";
import WebFrame from "@/components/WebFrame.vue";
import ImageContainer from "@/components/ImageContainer.vue";
import ControlScale from "@/components/ControlScale.vue";

const images = ref<URL[]>([]);
// window.onbeforeunload = function confirmExit() { return " " };
</script>

<template>
  <div class="flex">
    <div
      class="fixed inset-y-0 overflow-hidden bg-white transition-[width] duration-300 ease-in-out"
    >
      <WebFrame @insert_images="images = $event" />
    </div>
    <ControlScale>
      <template #default="{ scale, wheel_event, ref }">
        <ImageContainer
          class="ml-auto"
          :images="images"
          :ref="ref"
          :scale="scale"
          @wheel.ctrl.prevent="wheel_event"
        />
      </template>
    </ControlScale>
  </div>
</template>
