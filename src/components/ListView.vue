<script setup>
import { ref, onMounted, onUnmounted } from "vue";
import { monitorForElements } from "@atlaskit/pragmatic-drag-and-drop/element/adapter";
import { LIST_COLUMNS } from "../contstant.js";
import ListItem from "./ListItem.vue";
import { extractClosestEdge } from "@atlaskit/pragmatic-drag-and-drop-hitbox/closest-edge"; // 新
import { getReorderDestinationIndex } from "@atlaskit/pragmatic-drag-and-drop-hitbox/util/get-reorder-destination-index"; // 新
import { reorder } from "@atlaskit/pragmatic-drag-and-drop/reorder"; // 新

const items = ref(LIST_COLUMNS);

const reorderItem = ({ startIndex, finishIndex }) => {
  // 调用重新排序功能，得到一个新的卡数组
  // 包含被移动卡的新位置
  items.value = reorder({
    list: items.value,
    startIndex,
    finishIndex,
  });
};

onMounted(() => {
  const cleanup = monitorForElements({
    onDrop: ({ source, location }) => {
      // 如果当前位置没有拖放目标，则提前返回
      const destination = location.current.dropTargets.length;
      if (!destination) {
        return;
      }

      // 检查拖动源是否为 item，以处理 item 的特定逻辑
      if (source.data.type === "item") {
        // 获取被拖动 item 的 ID
        const draggedItemId = source.data.itemId;

        // 获取被拖动 item 的索引
        const draggedItemIndex = items.value.findIndex((item) => item.id === draggedItemId);

        // 获取目标 item 的数据
        const [destinationItemRecord] = location.current.dropTargets;

        // 获取目标 item 的索引
        const indexOfTarget = items.value.findIndex((item) => item.id === destinationItemRecord.data.itemId);

        // 确定目标 item 的最近边缘：顶部或底部
        const closestEdgeOfTarget = extractClosestEdge(destinationItemRecord.data);

        // 计算被拖动卡片的目标索引
        const destinationIndex = getReorderDestinationIndex({
          startIndex: draggedItemIndex,
          indexOfTarget,
          closestEdgeOfTarget,
          axis: "vertical",
        });

        // 重新排列
        reorderItem({
          startIndex: draggedItemIndex,
          finishIndex: destinationIndex,
        });
      }
    },
  });

  onUnmounted(() => {
    cleanup();
  });
});
</script>

<template>
  <div class="list">
    <list-item v-for="item in items" :key="item.id" :item="item"></list-item>
  </div>
</template>
