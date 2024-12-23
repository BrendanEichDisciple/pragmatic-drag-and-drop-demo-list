<script setup>
import { reactive } from "vue";

const { edge, gap } = defineProps({
  edge: {
    type: String,
    required: true,
  },
  gap: {
    type: String,
    required: true,
  },
});

const classObject = reactive({
  "edge-top": edge === "top",
  "edge-bottom": edge === "bottom",
});

const styleObject = reactive({
  "--gap": gap,
});
</script>

<template>
  <div class="drop-indicator" :class="classObject" :style="styleObject"></div>
</template>
