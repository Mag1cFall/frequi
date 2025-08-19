<script setup lang="ts">
import type { AllProfitStats } from '@/types';

const props = defineProps<{
  profitAll: AllProfitStats;
  stakeCurrency: string;
  stakeCurrencyDecimals: number;
}>();

const profit = computed(() => {
  if (!props.profitAll?.short) {
    return props.profitAll.all;
  }
  return props.profitAll[selectedOption.value];
});

const profitItems = computed(() => {
  if (!profit.value) return [];
  return [
    {
      metric: '已平仓交易投资回报率',
      value: profit.value.profit_closed_coin
        ? `${formatPriceCurrency(
            profit.value.profit_closed_coin,
            props.stakeCurrency,
            props.stakeCurrencyDecimals,
          )} (${formatPercent(profit.value.profit_closed_ratio_mean, 2)})`
        : 'N/A',
      // (&sum; ${formatPercent(profit.value.profit_closed_ratio_sum,  2,)})`
    },
    {
      metric: '所有交易投资回报率',
      value: profit.value.profit_all_coin
        ? `${formatPriceCurrency(
            profit.value.profit_all_coin,
            props.stakeCurrency,
            props.stakeCurrencyDecimals,
          )} (${formatPercent(profit.value.profit_all_ratio_mean, 2)})`
        : 'N/A',
      //  (&sum; ${formatPercent(profit.value.profit_all_ratio_sum,2,)})`
    },

    {
      metric: '总交易次数',
      value: `${profit.value.trade_count ?? 0}`,
    },
    {
      metric: '机器人启动于',
      value: profit.value.bot_start_timestamp,
      isTs: true,
    },
    {
      metric: '首笔交易开仓于',
      value: profit.value.first_trade_timestamp,
      isTs: true,
    },
    {
      metric: '最近交易开仓于',
      value: profit.value.latest_trade_timestamp,
      isTs: true,
    },
    {
      metric: '胜 / 负',
      value: `${profit.value.winning_trades ?? 0} / ${profit.value.losing_trades ?? 0}`,
    },
    {
      metric: '胜率',
      value: `${profit.value.winrate ? formatPercent(profit.value.winrate) : 'N/A'}`,
    },
    {
      metric: '期望值 (比率)',
      value: `${profit.value.expectancy ? profit.value.expectancy.toFixed(2) : 'N/A'} (${
        profit.value.expectancy_ratio ? profit.value.expectancy_ratio.toFixed(2) : 'N/A'
      })`,
    },
    {
      metric: '平均持仓时间',
      value: `${profit.value.avg_duration ?? 'N/A'}`,
    },
    {
      metric: '表现最佳',
      value: profit.value.best_pair
        ? `${profit.value.best_pair}: ${formatPercent(profit.value.best_pair_profit_ratio, 2)}`
        : 'N/A',
    },
    {
      metric: '交易量',
      value: `${formatPriceCurrency(
        profit.value.trading_volume ?? 0,
        props.stakeCurrency,
        props.stakeCurrencyDecimals,
      )}`,
    },
    {
      metric: '盈利因子',
      value: `${profit.value.profit_factor ? profit.value.profit_factor.toFixed(2) : 'N/A'}`,
    },
    {
      metric: '最大回撤',
      value: `${profit.value.max_drawdown ? formatPercent(profit.value.max_drawdown, 2) : 'N/A'} (${
        profit.value.max_drawdown_abs
          ? formatPriceCurrency(
              profit.value.max_drawdown_abs,
              props.stakeCurrency,
              props.stakeCurrencyDecimals,
            )
          : 'N/A'
      }) ${
        profit.value.max_drawdown_start_timestamp && profit.value.max_drawdown_end_timestamp
          ? '从 ' +
            timestampms(profit.value.max_drawdown_start_timestamp) +
            ' 到 ' +
            timestampms(profit.value.max_drawdown_end_timestamp)
          : ''
      }`,
    },
    {
      metric: '当前回撤',
      value: `${profit.value.current_drawdown ? formatPercent(profit.value.current_drawdown, 2) : 'N/A'} (${
        profit.value.current_drawdown_abs
          ? formatPriceCurrency(
              profit.value.current_drawdown_abs,
              props.stakeCurrency,
              props.stakeCurrencyDecimals,
            )
          : 'N/A'
      }) ${
        profit.value.current_drawdown_start_timestamp
          ? '从 ' + timestampms(profit.value.current_drawdown_start_timestamp) + ' 开始'
          : ''
      }`,
    },
  ];
});

const selectedOption = ref('all');
const options = [
  { value: 'all', text: '全部' },
  { value: 'long', text: '做多' },
  { value: 'short', text: '做空' },
];
</script>

<template>
  <div>
    <div v-if="profitAll?.long && profitAll?.short" class="flex justify-between items-center">
      <span>利润统计</span>
      <SelectButton
        v-model="selectedOption"
        :options="options"
        option-label="text"
        option-value="value"
        :allow-empty="false"
        size="small"
      ></SelectButton>
      <span>交易</span>
    </div>

    <DataTable class="text-start" small borderless :value="profitItems">
      <Column field="metric" header="指标"></Column>
      <Column field="value" header="值">
        <template #body="{ data }">
          <DateTimeTZ v-if="data.isTs" :date="data.value" show-timezone />
          <template v-else>
            {{ data.value }}
          </template>
        </template>
      </Column>
    </DataTable>
  </div>
</template>
