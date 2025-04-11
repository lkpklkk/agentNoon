<template>
  <div class="max-w-max mx-auto flex flex-col items-center">
    <svg
      v-if="expanded && childRefs.length"
      class="absolute pointer-events-none z-0"
      :width="svgWidth"
      :height="svgHeight"
      :viewBox="`${svgX} ${svgY} ${svgWidth} ${svgHeight}`"
    >
      <path
        v-for="(childRef, index) in childRefs"
        :key="index"
        :d="generatePath(getcenterBottom(cardRef), getcenterTop(childRef))"
        stroke="gray"
        stroke-width="1"
        fill="transparent"
      />
    </svg>
    <div
      ref="cardRef"
      class="border rounded-3xl p-4 bg-white shadow w-64 h-100 dark:bg-neutral-800 dark:border-blue-500"
    >
      <div
        class="flex items-start justify-between mb-2 flex-col h-full pl-4 pr-4"
      >
        <div>
          <h2 class="font-semibold text-gray-500 dark:text-neutral-50">
            {{ node.data['Name'] }}
          </h2>
          <p class="text-sm text-gray-500 dark:text-gray-100">
            {{ node.data['Job Title'] }}
          </p>
          <p class="text-sm text-gray-500 dark:text-gray-300">
            {{ node.data['level'] }}
          </p>
          <p class="text-sm text-gray-500 dark:text-gray-300">
            TotalCost: {{ formatNumber(node.data['TotalCost']) }}
          </p>
          <p class="text-sm text-gray-500 dark:text-gray-300">
            ManagementCost: {{ formatNumber(node.data['ManagementCost']) }}
          </p>
          <p class="text-sm text-gray-500 dark:text-gray-300">
            ICCost: {{ formatNumber(node.data['ICCost']) }}
          </p>
          <p class="text-sm text-gray-500 dark:text-gray-300">
            ICRatio: {{ node.data['ICRatio'] }}
          </p>
          <p class="text-sm text-gray-500 dark:text-gray-300">
            ManagementRatio: {{ node.data['ManagementRatio'] }}
          </p>
        </div>
        <div
          class="w-full border-t border-gray-600 my-2 border-0 dark:border-gray-300"
        ></div>
        <div class="flex items-start justify-between flex-row w-full">
          <div class="flex justify-center items-center flex-col">
            <h2 class="bold text-xl text-gray-500 dark:text-gray-300">
              Standing
            </h2>

            <div class="text-xl">
              {{ node.data['Performance']?.split(' ')[0] || '' }}
            </div>
          </div>
          <div class="flex justify-center items-center flex-col">
            <h2 class="bold text-xl text-gray-500 dark:text-gray-300">
              Total comp
            </h2>
            <div class="text-xl text-gray-500 dark:text-gray-300">
              {{ formatNumber(node.data['Salary'] + node.data['Bonus']) }}
            </div>
            <!-- <div>{{ node.data['ExpensivenessScale'] }}</div> -->
            <div>{{ '💵'.repeat(node.data['ExpensivenessScale']) }}</div>
          </div>
        </div>
        <div class="flex items-center justify-center flex-row w-full">
          <button
            v-if="node.children && node.children.length > 0"
            @click="toggleExpand"
            class="border rounded-md p-1 text-blue-600 hover:text-blue-800 text-sm transition-opacity duration-300 ease-in-out"
          >
            <span key="text" class="mr-1">{{
              expanded ? 'Collapse' : 'Expand'
            }}</span>
            <span class=""> {{ expanded ? '⬆️' : '⬇️' }}</span>
          </button>
        </div>
      </div>
    </div>

    <transition name="fade">
      <div
        v-if="expanded"
        ref="childrenWrapper"
        class="flex flex-row flex-nowrap pt-10"
      >
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
import { ref, watch, nextTick, inject, onMounted, onUnmounted } from 'vue';
import EmployeeCard from './EmployeeCard.vue';

const props = defineProps({
  node: {
    type: Object,
    required: true,
  },
});

const expanded = ref(false);
const cardRef = ref(null);
const childRefs = ref([]);
const childrenWrapper = ref(null);
const offsetXRef = inject('offsetXRef');
const offsetYRef = inject('offsetYRef');
const scaleRef = inject('scaleRef');
const panContainerRef = inject('panContainerRef');
const svgWidth = ref(0);
const svgHeight = ref(0);
const svgX = ref(0);
const svgY = ref(0);

function toggleExpand() {
  expanded.value = !expanded.value;
}
function updateChildRefs() {
  if (!childrenWrapper.value) return;
  childRefs.value = Array.from(childrenWrapper.value.children);
  childRefs.value = [...childRefs.value];
}
function updateSVGWindow() {
  if (!cardRef.value) return;
  if (!childrenWrapper.value) return;
  const cardRect = cardRef.value.getBoundingClientRect();
  const childrenRect = childrenWrapper.value.getBoundingClientRect();

  svgX.value =
    (Math.min(cardRect.left, childrenRect.left) - offsetXRef.value) /
    scaleRef.value;
  svgY.value =
    (Math.min(cardRect.top, childrenRect.top) - offsetYRef.value) /
    scaleRef.value;
  svgWidth.value =
    Math.max(cardRect.width, childrenRect.width) / scaleRef.value;
  svgHeight.value = (cardRect.height + childrenRect.height) / scaleRef.value;
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
function getcenterTop(e) {
  if (!e) return { x: 0, y: 0 };
  const rect = e.getBoundingClientRect();

  return {
    x: (rect.left + rect.width / 2 - offsetXRef.value) / scaleRef.value,
    y: (rect.top - offsetYRef.value) / scaleRef.value,
  };
}

function getcenterBottom(e) {
  if (!e) return { x: 0, y: 0 };
  const rect = e.getBoundingClientRect();

  return {
    x: (rect.left + rect.width / 2 - offsetXRef.value) / scaleRef.value,
    y: (rect.top + rect.height - offsetYRef.value) / scaleRef.value,
  };
}

function generatePath(from, to) {
  const controlPointX1 = from.x;
  const controlPointY1 = from.y + 20;
  const controlPointX2 = to.x;
  const controlPointY2 = to.y - 50;

  return `M${from.x},${from.y} C${controlPointX1},${controlPointY1} ${controlPointX2},${controlPointY2} ${to.x},${to.y}`;
}
watch(expanded, (newVal) => {
  window.dispatchEvent(new Event('redrawLines'));
});

onMounted(() => {
  // wait for animation to finish
  const redrawLines = () => {
    setTimeout(() => {
      updateChildRefs();
      updateSVGWindow();
    }, 300);
  };

  window.addEventListener('redrawLines', redrawLines);

  onUnmounted(() => {
    window.removeEventListener('redrawLines', redrawLines);
  });
});
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
