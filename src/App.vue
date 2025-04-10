<script setup>
import { onMounted, ref } from 'vue';
import EmployeeCard from './components/EmployeeCard.vue';
import * as d3 from 'd3';

const root = ref(null);

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
      ManagementCostRatio: 0,
    };
  });
  const hierachy = d3
    .stratify()
    .id((d) => d['Employee Id'])
    .parentId((d) => d['Manager'])(updated_data);
  hierachy.eachAfter((node) => {
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
  root.value = hierachy;
});
</script>

<template>
  <div class="flex flex-col items-center justify-center">
    <img
      src="./assets/agentnoon.png"
      alt="Agent Noon"
      class="w-1/2 max-w-sm sm:w-1/3 md:w-1/4 lg:w-1/5 object-contain"
    />
    <EmployeeCard v-if="root" :node="root"></EmployeeCard>
  </div>
</template>
