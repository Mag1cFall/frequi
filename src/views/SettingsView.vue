<script setup lang="ts">
import { FtWsMessageTypes } from '@/types/wsMessageTypes';

const settingsStore = useSettingsStore();
const colorStore = useColorStore();
const layoutStore = useLayoutStore();

const timezoneOptions = ['UTC', Intl.DateTimeFormat().resolvedOptions().timeZone];
const openTradesOptions = [
  { value: OpenTradeVizOptions.showPill, text: 'Show pill in icon' },
  { value: OpenTradeVizOptions.asTitle, text: 'Show in title' },
  { value: OpenTradeVizOptions.noOpenTrades, text: "Don't show open trades in header" },
];
const colorPreferenceOptions = [
  { value: ColorPreferences.GREEN_UP, text: 'Green Up/Red Down' },
  { value: ColorPreferences.RED_UP, text: 'Green Down/Red Up' },
];

const resetDynamicLayout = () => {
  layoutStore.resetTradingLayout();
  layoutStore.resetDashboardLayout();
  showAlert('Layouts have been reset.');
};
</script>

<template>
  <Card class="mx-auto mt-3 p-4 max-w-4xl">
    <template #title>FreqUI 設定</template>
    <template #content>
      <div class="flex flex-col gap-4 text-start dark:text-surface-300">
        <p class="text-left">UI 版本: {{ settingsStore.uiVersion }}</p>

        <div class="border border-surface-400 rounded-sm p-4 space-y-4">
          <h4 class="text-xl font-semibold">UI 設定</h4>

          <BaseCheckbox v-model="layoutStore.layoutLocked" class="space-y-1">
            鎖定動態佈局
            <template #hint>
              鎖定動態佈局，使其無法移動。也可以從頂部的導航欄設定。
            </template>
          </BaseCheckbox>

          <div class="flex flex-row items-center gap-2 space-y-2">
            <Button severity="secondary" size="small" class="mb-0" @click="resetDynamicLayout"
              >重置佈局</Button
            >
            <small class="block text-surface-600 dark:text-surface-400"
              >將動態佈局重置為預設值。</small
            >
          </div>

          <Divider />

          <div class="space-y-1">
            <label class="block text-sm">在頁首顯示當前交易</label>
            <Select
              v-model="settingsStore.openTradesInTitle"
              :options="openTradesOptions"
              option-label="text"
              option-value="value"
              size="small"
              class="w-full"
            />
            <small class="text-surface-600 dark:text-surface-400"
              >決定是否要將當前交易視覺化</small
            >
          </div>

          <div class="space-y-1">
            <label class="block text-sm">UTC 時區</label>
            <Select
              v-model="settingsStore.timezone"
              :options="timezoneOptions"
              class="w-full"
              size="small"
            />
            <small class="text-surface-600 dark:text-surface-400"
              >選擇時區（建議使用 UTC，因為交易所通常使用 UTC）</small
            >
          </div>

          <BaseCheckbox v-model="settingsStore.backgroundSync" class="space-y-1">
            背景同步
            <template #hint> 選擇其他機器人時，保持背景同步運行。 </template>
          </BaseCheckbox>

          <BaseCheckbox v-model="settingsStore.confirmDialog" class="space-y-1">
            顯示平倉確認對話框
            <template #hint>強制平倉時使用確認對話框。</template>
          </BaseCheckbox>

          <BaseCheckbox v-model="settingsStore.multiPaneButtonsShowText" class="space-y-1">
            在綜合面版按鈕上顯示文字
            <template #hint
              >在綜合面版按鈕上顯示文字。如果停用，只會顯示圖示。</template
            >
          </BaseCheckbox>
        </div>

        <div class="border border-surface-400 rounded-sm p-4 space-y-4">
          <h4 class="text-lg font-semibold">圖表設定</h4>

          <div class="space-y-1">
            <label class="block text-sm">圖表刻度位置</label>
            <div class="flex gap-4">
              <div class="flex items-center">
                <RadioButton v-model="settingsStore.chartLabelSide" value="left" size="small" />
                <label class="ml-2">左側</label>
              </div>
              <div class="flex items-center">
                <RadioButton v-model="settingsStore.chartLabelSide" value="right" size="small" />
                <label class="ml-2">右側</label>
              </div>
            </div>
            <small class="text-surface-600 dark:text-surface-400">
              刻度應該顯示在右側還是左側？
            </small>
          </div>

          <BaseCheckbox v-model="settingsStore.useHeikinAshiCandles" class="space-y-1">
            使用 Heikin Ashi K線
            <template #hint>在圖表中使用 Heikin Ashi K線</template>
          </BaseCheckbox>

          <BaseCheckbox v-model="settingsStore.useReducedPairCalls" class="space-y-1">
            僅請求必要的欄位
            <template #hint
              >可以減少大型數據幀的傳輸大小。如果繪圖設定變更，可能需要額外的呼叫。</template
            >
          </BaseCheckbox>

          <div>
            <p>預設顯示的K線數量（預設為 250）</p>
            <div class="flex flex-row gap-5 w-full items-center">
              <Slider
                v-model="settingsStore.chartDefaultCandleCount"
                class="flex-1"
                :step="50"
                :min="100"
                :max="2000"
              />
              <InputNumber
                v-model="settingsStore.chartDefaultCandleCount"
                :step="50"
                :min="100"
                :max="2000"
                size="small"
              />
            </div>
          </div>

          <div class="space-y-1">
            <label class="block">K線顏色偏好</label>
            <div class="flex flex-row gap-5 items-center">
              <div
                v-for="option in colorPreferenceOptions"
                :key="option.value"
                class="flex items-center"
              >
                <RadioButton
                  v-model="colorStore.colorPreference"
                  :value="option.value"
                  :input-id="`input-id${option.value}`"
                  size="small"
                  @change="colorStore.updateProfitLossColor"
                />
                <label :for="`input-id${option.value}`" class="ml-2 flex items-center">
                  <span class="mr-2">{{ option.text }}</span>
                  <i-mdi-arrow-up-thin
                    :color="
                      option.value === ColorPreferences.GREEN_UP
                        ? colorStore.colorProfit
                        : colorStore.colorLoss
                    "
                    class="-ml-2"
                  />
                  <i-mdi-arrow-down-thin
                    :color="
                      option.value === ColorPreferences.GREEN_UP
                        ? colorStore.colorLoss
                        : colorStore.colorProfit
                    "
                    class="-ml-2"
                  />
                </label>
              </div>
            </div>
          </div>
        </div>

        <div class="border rounded-sm p-4 space-y-4">
          <h4 class="text-lg font-semibold">通知設定</h4>
          <div class="space-y-2">
            <BaseCheckbox v-model="settingsStore.notifications[FtWsMessageTypes.entryFill]">
              進場通知
            </BaseCheckbox>
            <BaseCheckbox v-model="settingsStore.notifications[FtWsMessageTypes.exitFill]">
              出場通知
            </BaseCheckbox>
            <BaseCheckbox v-model="settingsStore.notifications[FtWsMessageTypes.entryCancel]">
              取消進場通知
            </BaseCheckbox>
            <BaseCheckbox v-model="settingsStore.notifications[FtWsMessageTypes.exitCancel]">
              取消出場通知
            </BaseCheckbox>
          </div>
        </div>

        <div class="border rounded-sm p-4 space-y-4">
          <h4 class="text-lg font-semibold">回測設定</h4>
          <div>
            <label for="backtestMetrics" class="block text-sm">回測指標</label>
            <MultiSelect
              id="backtestMetrics"
              v-model="settingsStore.backtestAdditionalMetrics"
              :options="availableBacktestMetrics"
              option-label="header"
              option-value="field"
              class="w-full"
              size="small"
              display="chip"
            />
            <small class="text-surface-600 dark:text-surface-400"
              >選擇應在每個交易對/標籤基礎上顯示哪些指標。</small
            >
          </div>
        </div>
      </div>
    </template>
  </Card>
</template>
