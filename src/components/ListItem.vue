<script setup>
const { item } = defineProps({
  item: {
    type: Object,
    required: true,
  },
});
</script>

<template>
  <div class="list-item">
    {{ item.id }} {{ item.label }}
  </div>
</template>
