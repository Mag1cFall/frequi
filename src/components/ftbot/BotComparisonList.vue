<script setup lang="ts">
import type { ProfitStats, ComparisonTableItems } from '@/types';

const botStore = useBotStore();

const allToggled = computed<boolean>({
  get: () => Object.values(botStore.botStores).every((i) => i.isSelected),
  set: (val) => {
    for (const botId in botStore.botStores) {
      botStore.botStores[botId].isSelected = val;
    }
  },
});

const tableItems = computed<ComparisonTableItems[]>(() => {
  const val: ComparisonTableItems[] = [];
  const summary: ComparisonTableItems = {
    botId: undefined,
    botName: '摘要',
    profitClosed: 0,
    profitClosedRatio: undefined,
    profitOpen: 0,
    profitOpenRatio: undefined,
    stakeCurrency: 'USDT',
    wins: 0,
    losses: 0,
  };

  Object.entries(botStore.allProfit).forEach(([k, v]: [k: string, v: ProfitStats]) => {
    const allStakes = botStore.allOpenTrades[k].reduce((a, b) => a + b.stake_amount, 0);
    const profitOpenRatio =
      botStore.allOpenTrades[k].reduce(
        (a, b) => a + (b.total_profit_ratio ?? b.profit_ratio) * b.stake_amount,
        0,
      ) / allStakes;
    const profitOpen = botStore.allOpenTrades[k].reduce(
      (a, b) => a + (b.total_profit_abs ?? b.profit_abs ?? 0),
      0,
    );

    // TODO: handle one inactive bot ...
    val.push({
      botId: k,
      botName: botStore.availableBots[k].botName || botStore.availableBots[k].botId,
      trades: `${botStore.allOpenTradeCount[k]} / ${
        botStore.allBotState[k]?.max_open_trades || 'N/A'
      }`,
      profitClosed: v.profit_closed_coin,
      profitClosedRatio: v.profit_closed_ratio || 0,
      stakeCurrency: botStore.allBotState[k]?.stake_currency || '',
      profitOpenRatio,
      profitOpen,
      wins: v.winning_trades,
      losses: v.losing_trades,
      balance: botStore.allBalance[k]?.total_bot ?? botStore.allBalance[k]?.total,
      stakeCurrencyDecimals: botStore.allBotState[k]?.stake_currency_decimals || 3,
      isDryRun: botStore.allBotState[k]?.dry_run,
      isOnline: botStore.botStores[k]?.isBotOnline,
    });
    if (v.profit_closed_coin !== undefined) {
      if (botStore.botStores[k].isSelected) {
        // Summary should only include selected bots
        summary.profitClosed += v.profit_closed_coin;
        summary.profitOpen += profitOpen;
        summary.wins += v.winning_trades;
        summary.losses += v.losing_trades;
        // summary.decimals = this.allBotState[k]?.stake_currency_decimals || summary.decimals;
        // This will always take the last bot's stake currency
        // And therefore may result in wrong values.
        summary.stakeCurrency = botStore.allBotState[k]?.stake_currency || summary.stakeCurrency;
      }
    }
  });
  val.push(summary);
  return val;
});
</script>

<template>
  <DataTable size="small" :value="tableItems">
    <Column field="botName" header="机器人">
      <template #body="{ data, field }">
        <div class="flex flex-row justify-between items-center">
          <div>
            <BaseCheckbox
              v-if="data.botId && botStore.botCount > 1"
              v-model="
                botStore.botStores[(data as unknown as ComparisonTableItems).botId ?? ''].isSelected
              "
              title="在仪表盘中显示此机器人"
              >{{ data[field] }}</BaseCheckbox
            >
            <BaseCheckbox
              v-if="!data.botId && botStore.botCount > 1"
              v-model="allToggled"
              title="切换所有机器人"
              class="font-bold"
              >{{ data[field] }}</BaseCheckbox
            >
            <span v-if="botStore.botCount <= 1">{{ data[field] }}</span>
          </div>
          <Badge
            v-if="data.isOnline && data.isDryRun"
            class="items-center bg-green-800 text-slate-200"
            severity="success"
            >模拟</Badge
          >
          <Badge v-if="data.isOnline && !data.isDryRun" class="items-center" severity="warning"
            >实盘</Badge
          >
          <Badge v-if="data.isOnline === false" class="items-center" severity="secondary"
            >离线</Badge
          >
        </div>
      </template>
    </Column>
    <Column field="trades" header="交易"> </Column>
    <Column header="持仓利润">
      <template #body="{ data }">
        <ProfitPill
          v-if="data.profitOpen && data.botId != '摘要'"
          :profit-ratio="(data as unknown as ComparisonTableItems).profitOpenRatio"
          :profit-abs="(data as unknown as ComparisonTableItems).profitOpen"
          :profit-desc="`总利润 (持仓及已实现) ${formatPercent(
            (data as ComparisonTableItems).profitOpenRatio ?? 0.0,
          )}`"
          :stake-currency="(data as ComparisonTableItems).stakeCurrency"
        />
      </template>
    </Column>
    <Column header="已实现利润">
      <template #body="{ data }">
        <ProfitPill
          v-if="data.profitClosed && data.botId != '摘要'"
          :profit-ratio="(data as ComparisonTableItems).profitClosedRatio"
          :profit-abs="(data as ComparisonTableItems).profitClosed"
          :stake-currency="(data as unknown as ComparisonTableItems).stakeCurrency"
        />
      </template>
    </Column>
    <Column field="balance" header="余额">
      <template #body="{ data }">
        <div v-if="data.balance">
          <span :title="(data as ComparisonTableItems).stakeCurrency"
            >{{
              formatPrice(
                (data as ComparisonTableItems).balance ?? 0,
                (data as ComparisonTableItems).stakeCurrencyDecimals,
              )
            }}
          </span>
          <span class="text-sm">{{
            ` ${data.stakeCurrency}${data.isDryRun ? ' (模拟)' : ''}`
          }}</span>
        </div>
      </template>
    </Column>
    <Column field="winVsLoss" header="盈/亏">
      <template #body="{ data }">
        <div v-if="data.losses !== undefined">
          <span class="text-profit">{{ data.wins }}</span> /
          <span class="text-loss">{{ data.losses }}</span>
        </div>
      </template>
    </Column>
  </DataTable>
</template>
