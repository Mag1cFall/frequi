<script setup lang="ts">
import type { PeriodicBreakdown } from '@/types';

const props = defineProps<{
  periodicBreakdown: PeriodicBreakdown;
}>();

const periodicBreakdownSelections = computed(() => {
  const res = [
    { value: 'day', text: '天' },
    { value: 'week', text: '周' },
    { value: 'month', text: '月' },
  ];
  if (props.periodicBreakdown.year) {
    res.push({ value: 'year', text: '年' });
  }

  return res;
});

const periodicBreakdownPeriod = ref<string>('month');
</script>

<template>
  <SelectButton
    v-model="periodicBreakdownPeriod"
    :options="periodicBreakdownSelections"
    size="small"
    :allow-empty="false"
    class="m-2"
    option-label="text"
    option-value="value"
  ></SelectButton>
  <DataTable size="small" stacked="sm" :value="periodicBreakdown[periodicBreakdownPeriod]">
    <Column field="date" header="日期"></Column>
    <Column field="trades" header="交易次数">
      <template #body="{ data, field }">
        {{ data[field] ?? 'N/A' }}
      </template>
    </Column>
    <Column field="profit_abs" header="总利润" :body="formatPrice">
      <template #body="{ data, field }">
        {{ data[field] ? data[field].toFixed(2) : 'N/A' }}
      </template>
    </Column>
    <Column field="profit_factor" header="盈利因子">
      <template #body="{ data, field }">
        {{ formatPrice(data[field], 2) }}
      </template>
    </Column>
    <Column field="wins" header="盈利次数"></Column>
    <Column field="draws" header="平局次数"></Column>
    <Column field="losses" header="亏损次数">
      <template #body="{ data }">
        {{ data.loses ?? data.losses ?? 'N/A' }}
      </template>
    </Column>
    <Column field="wins" header="胜率">
      <template #body="{ data }">
        {{
          ((data.wins / (data.wins + data.draws + (data.loses ?? data.losses))) * 100).toFixed(2) +
          '%'
        }}
      </template>
    </Column>
  </DataTable>
</template>
