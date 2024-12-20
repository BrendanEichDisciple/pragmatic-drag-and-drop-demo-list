<script setup>
import { ref } from "vue";
import { LIST_COLUMNS } from "../contstant.js";
import ListItem from "./ListItem.vue";

const items = ref(LIST_COLUMNS);
</script>

<template>
  <div class="list">
    <list-item v-for="item in items" :key="item.id" :item="item"></list-item>
  </div>
</template>
