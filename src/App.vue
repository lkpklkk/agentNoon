<script setup>
import { onMounted, ref, provide } from 'vue';
import EmployeeCard from './components/EmployeeCard.vue';
import * as d3 from 'd3';
const topPaddingPanContainer = 200;
const root = ref(null);
const canvasRef = ref(null);
const panContainerRef = ref(null);
const offsetX = ref(0);
const offsetY = ref(topPaddingPanContainer);
const scale = ref(1);
let isDragging = false;
let startX = 0;
let startY = 0;
let minTotalCost = 0;
let maxTotalCost = 0;

const isDark = ref(false);

provide('offsetXRef', offsetX);
provide('offsetYRef', offsetY);
provide('scaleRef', scale);
provide('panContainerRef', panContainerRef);

function clamp(value, min, max) {
  return Math.max(min, Math.min(value, max));
}

function onWheel(e) {
  if (e.altKey) {
    const scaleAmount = 0.1;
    const newScale = scale.value + (e.deltaY > 0 ? -scaleAmount : scaleAmount);
    scale.value = Math.max(0.1, Math.min(newScale, 10));
    window.dispatchEvent(new Event('redrawLines'));
  } else {
    offsetX.value -= e.deltaX;
    offsetY.value -= e.deltaY;

    const panContainer = panContainerRef.value;

    offsetX.value = clamp(
      offsetX.value,
      -panContainer.scrollWidth / 1.5,
      panContainer.scrollWidth / 1.5
    );
    offsetY.value = clamp(
      offsetY.value,
      ((-panContainer.scrollHeight - topPaddingPanContainer) * scale.value) /
        1.5,
      ((panContainer.scrollHeight + topPaddingPanContainer) * scale.value) / 1.5
    );
  }
}

function onMouseDown(e) {
  isDragging = true;
  startX = e.clientX - offsetX.value;
  startY = e.clientY - offsetY.value;
}

function onMouseMove(e) {
  if (!isDragging) return;
  offsetX.value = e.clientX - startX;
  offsetY.value = e.clientY - startY;

  const panContainer = panContainerRef.value;

  offsetX.value = clamp(
    offsetX.value,
    (-panContainer.scrollWidth * scale.value) / 1.5,
    (panContainer.scrollWidth * scale.value) / 1.5
  );
  offsetY.value = clamp(
    offsetY.value,
    ((-panContainer.scrollHeight - topPaddingPanContainer) * scale.value) / 1.5,
    ((panContainer.scrollHeight + topPaddingPanContainer) * scale.value) / 1.5
  );
}

function onMouseUp() {
  isDragging = false;
}

onMounted(async () => {
  // const employee_data = await d3.csv('/data/test.csv');

  // const hierachy = d3
  //   .stratify()
  //   .id((d) => d.employeeId)
  //   .parentId((d) => d.managerId)(employee_data);

  const employee_data = await d3.csv('/data/Giga Corp 40k Data.csv');
  const updated_data = employee_data.map((row) => {
    return {
      ...row,
      Salary: parseInt(row['Salary'], 10) || 0,
      Bonus: parseInt(row['Bonus'], 10) || 0,
      ManagementCost: 0,
      ICCost: 0,
      TotalCost: 0,
      ManagementRatio: 0,
      ICRatio: 0,
      ExpensivenessScale: 0,
    };
  });
  const hierachy = d3
    .stratify()
    .id((d) => d['Employee Id'])
    .parentId((d) => d['Manager'])(updated_data);
  let maxSalary = 1;
  let minSalary = 10000000000;
  hierachy.eachAfter((node) => {
    maxSalary = Math.max(maxSalary, node.data['Salary'] + node.data['Bonus']);
    minSalary = Math.min(minSalary, node.data['Salary'] + node.data['Bonus']);
    if (node.children) {
      node.children.forEach((child) => {
        node.data['TotalCost'] += child.data['TotalCost'];
        node.data['ICCost'] += child.data['ICCost'];
      });
      node.data['TotalCost'] += node.data['Salary'] + node.data['Bonus'];
      node.data['ManagementCost'] =
        node.data['TotalCost'] - node.data['ICCost'];
    } else {
      node.data['ICCost'] = node.data['Salary'] + node.data['Bonus'];
      node.data['TotalCost'] = node.data['Salary'] + node.data['Bonus'];
    }
  });

  // just gonna do rank next time I guess
  hierachy.eachAfter((node) => {
    node.data['ManagementRatio'] = (
      node.data['ManagementCost'] / node.data['TotalCost']
    ).toFixed(1);
    node.data['ICRatio'] = (
      node.data['ICCost'] / node.data['TotalCost']
    ).toFixed(1);
    const logExpensiveness =
      node.data['Salary'] + node.data['Bonus'] - minSalary > 0
        ? Math.log(node.data['Salary'] + node.data['Bonus'] - minSalary) /
          Math.log(maxSalary - minSalary)
        : 0;
    node.data['ExpensivenessScale'] = Math.floor(logExpensiveness * 4 + 1);
  });
  root.value = hierachy;

  const canvas = canvasRef.value;
  canvas.addEventListener('mousedown', onMouseDown);
  canvas.addEventListener('mousemove', onMouseMove);
  canvas.addEventListener('mouseup', onMouseUp);
  canvas.addEventListener('mouseleave', onMouseUp);
});

function toggleTheme() {
  document.documentElement.classList.toggle('dark');
  isDark.value = !isDark.value;
}
</script>

<template>
  <div class="relative w-screen h-screen overflow-hidden">
    <div
      ref="canvasRef"
      class="absolute top-0 left-0 w-full h-full overflow-hidden bg-dotted dark:bg-dotted-dark"
      @wheel.passive="onWheel"
    >
      <div
        ref="panContainerRef"
        class="relative"
        :style="{
          transform: `translate(${offsetX}px, ${offsetY}px) scale(${scale})`,
        }"
      >
        <EmployeeCard v-if="root" :node="root" />
      </div>
    </div>
    <div class="w-full h-1/12 flex justify-left items-center mt-7 pl-10 pt-10">
      <div
        class="rounded-full glass-header w-80 h-20 flex justify-center items-center flex-nowrap"
      >
        <img
          src="./assets/agentnoon.png"
          alt="Agent Noon"
          class="animate-pulse w-40 ml-7 mr-7"
        />
        <div class="h-full flex justify-center items-center">
          <button
            @click="toggleTheme"
            class="relative inline-flex h-8 w-16 items-center rounded-full bg-gray-300 dark:bg-neutral-700 transition-colors duration-300 focus:outline-none"
          >
            <span
              :class="[
                'absolute left-1 top-1 h-6 w-6 rounded-full bg-white transition-transform duration-300',
                isDark ? 'translate-x-8' : 'translate-x-0',
              ]"
            />
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<style>
.glass-header {
  background: rgba(255, 255, 255, 0.2); /* Semi-transparent white background */
  backdrop-filter: blur(10px); /* Blur effect */
  -webkit-backdrop-filter: blur(10px); /* Safari support */
  border: 1px solid rgba(255, 255, 255, 0.3); /* Light border for the glass effect */
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1); /* Subtle shadow for depth */
  z-index: 10; /* Ensure it stays on top */
}
.bg-dotted {
  @apply bg-[#f9f9f9] bg-[radial-gradient(#ccc_1px,transparent_1px)] bg-[length:20px_20px];
}

.dark .bg-dotted {
  @apply bg-[#1a1a1a] bg-[radial-gradient(#444_1px,transparent_1px)];
}
</style>
