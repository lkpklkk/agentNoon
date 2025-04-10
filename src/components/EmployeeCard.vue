<template>
  <div class="max-w-max mx-auto flex flex-col items-center">
    <div class="border rounded-md p-4 mb-4 bg-white shadow w-64 h-64">
      <div class="flex items-center justify-between mb-2">
        <div>
          <h2 class="font-semibold">{{ node.data['Name'] }}</h2>
          <p class="text-sm text-gray-500">{{ node.data['Job Title'] }}</p>
          <p class="text-sm text-gray-500">
            {{ node.data['level'] }}
          </p>
          <p class="text-sm text-gray-500">
            TotalCost:{{ formatNumber(node.data['TotalCost']) }}
          </p>
          <p class="text-sm text-gray-500">
            ManagementCost {{ formatNumber(node.data['ManagementCost']) }}
          </p>
          <p class="text-sm text-gray-500">
            ICCost: {{ formatNumber(node.data['ICCost']) }}
          </p>
        </div>

        <button
          v-if="node.children && node.children.length > 0"
          @click="toggleExpand"
          class="border rounded-md p-1 text-blue-600 hover:text-blue-800 text-sm ml-2"
        >
          {{ expanded ? 'Collapse' : 'Expand' }}
        </button>
      </div>
    </div>

    <transition name="fade">
      <div v-if="expanded" class="flex flex-row flex-nowrap pl-4 mt-2">
        <employee-card
          class="ml-1 mr-1"
          v-for="child in node.children"
          :key="child.id"
          :node="child"
        />
      </div>
    </transition>
  </div>
</template>

<script setup>
import { ref } from 'vue';
import EmployeeCard from './EmployeeCard.vue';

const props = defineProps({
  node: {
    type: Object,
    required: true,
  },
});
console.log(props.node);
const expanded = ref(false);

function toggleExpand() {
  expanded.value = !expanded.value;
}

function formatNumber(value) {
  if (value >= 1_000_000_000) {
    return `${(value / 1_000_000_000).toFixed(1)}bn`;
  } else if (value >= 1_000_000) {
    return `${(value / 1_000_000).toFixed(1)}m`;
  } else if (value >= 1_000) {
    return `${(value / 1_000).toFixed(1)}k`;
  }
  return value.toString();
}
</script>

<style scoped>
.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.2s;
}
.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}
</style>
