<script setup>
defineProps({
  item: {
    type: Object,
    required: true,
  },
});
</script>

<template>
  <div class="list-item">
    {{ item.label }}
  </div>
</template>
