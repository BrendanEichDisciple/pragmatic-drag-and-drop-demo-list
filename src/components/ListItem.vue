<script setup>
import { ref, onMounted, onUnmounted } from "vue";
import invariant from "tiny-invariant";
import { draggable, dropTargetForElements } from "@atlaskit/pragmatic-drag-and-drop/element/adapter";
import { combine } from "@atlaskit/pragmatic-drag-and-drop/combine"; // 新
import { attachClosestEdge, extractClosestEdge } from "@atlaskit/pragmatic-drag-and-drop-hitbox/closest-edge"; // 新
import DropIndicator from "./DropIndicator.vue"; // 新

const { item } = defineProps({
  item: {
    type: Object,
    required: true,
  },
});

const itemRef = ref(null); // 创建一个引用
const isDragging = ref(false); // 创建一个拖动状态
const closestEdge = ref(null); // 新

onMounted(() => {
  const itemEl = itemRef.value;
  invariant(itemEl); // 确保引用存在

  // 合并 draggable 和 dropTargetForElements 清理函数
  // 返回单一清理函数
  const cleanup = combine(
    draggable({
      element: itemEl, // 为元素设置可拖动行为
      getInitialData: () => ({ type: "item", itemId: item.id }), // 开始拖动时将数据附加到可拖动项目上
      onDragStart: () => (isDragging.value = true), // 开始拖动时设置拖动状态
      onDrop: () => (isDragging.value = false), // 结束拖动时清除拖动状态
    }),
    // 添加 dropTargetForElements，使 item 元素成为放置目标
    dropTargetForElements({
      element: itemEl,
      getData: ({ input, element }) => {
        // 将 item 数据附加到拖放目标
        const data = { type: "item", itemId: item.id };

        // 将最靠近的边缘（顶部或底部）附加到数据对象
        // 该数据将用于确定相对于目标 item下放 item 的位置
        return attachClosestEdge(data, {
          input,
          element,
          allowedEdges: ["top", "bottom"],
        });
      },
      getIsSticky: () => true, // 使投放目标具有 "粘性"
      // 新
      onDragEnter: (args) => {
        // 当可拖动项目进入下拉区域时，更新最近的边缘
        if (args.source.data.itemId !== item.id) {
          closestEdge.value = extractClosestEdge(args.self.data);
        }
      },
      onDrag: (args) => {
        // 在拖动到下拉区域时，持续更新最近的边缘
        if (args.source.data.itemId !== item.id) {
          closestEdge.value = extractClosestEdge(args.self.data);
        }
      },
      onDragLeave: () => {
        // 当可拖动项目离开下拉区域时，重置最近的边缘
        closestEdge.value = null;
      },
      onDrop: (args) => {
        // 在拖放可拖动项目时重置最近的边缘
        closestEdge.value = null;
      },
    })
  );

  onUnmounted(() => {
    cleanup(); // 清理拖动行为
  });
});
</script>

<template>
  <!-- 当拖动时，isDragging 为 true，添加 .dragging 类 -->
  <div class="list-item" ref="itemRef" :class="{ dragging: isDragging }">
    {{ item.id }} {{ item.label }}
    <!-- 如果有最近边缘，则渲染 DropIndicator -->
    <drop-indicator v-if="closestEdge" :edge="closestEdge" gap="8px" />
  </div>
</template>
