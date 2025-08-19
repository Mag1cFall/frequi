forceexit
<script setup lang="ts">
import type { MsgBoxObject } from '@/components/general/MessageBox.vue';
import type MessageBox from '@/components/general/MessageBox.vue';

import type { ForceSellPayload } from '@/types';

import ForceEntryForm from './ForceEntryForm.vue';

const botStore = useBotStore();
const forceEnter = ref<boolean>(false);
const msgBox = ref<typeof MessageBox>();

const isRunning = computed((): boolean => {
  return botStore.activeBot.botState?.state === 'running';
});

const handleStopBot = () => {
  const msg: MsgBoxObject = {
    title: '停止机器人',
    message: '确定要停止机器人运行循环吗？',
    accept: () => {
      botStore.activeBot.stopBot();
    },
  };
  msgBox.value?.show(msg);
};

const handleStopBuy = () => {
  const msg: MsgBoxObject = {
    title: '暂停 - 停止进场',
    message:
      'Freqtrade 将继续处理现有持仓，但不会建立新的仓位或增加仓位大小。',
    accept: () => {
      botStore.activeBot.stopBuy();
    },
  };
  msgBox.value?.show(msg);
};

const handleReloadConfig = () => {
  const msg: MsgBoxObject = {
    title: '重新加载',
    message: '重新加载配置（包括策略）吗？',
    accept: () => {
      console.log('reload...');
      botStore.activeBot.reloadConfig();
    },
  };
  msgBox.value?.show(msg);
};

const handleForceExit = () => {
  const msg: MsgBoxObject = {
    title: '全部强制平仓',
    message: '真的要强制平仓所有交易吗？',
    accept: () => {
      const payload: ForceSellPayload = {
        tradeid: 'all',
        // TODO: support ordertype (?)
      };
      botStore.activeBot.forceexit(payload);
    },
  };
  msgBox.value?.show(msg);
};
</script>

<template>
  <div class="flex flex-row gap-1">
    <Button
      size="large"
      severity="secondary"
      :disabled="!botStore.activeBot.isTrading || isRunning"
      title="开始交易"
      @click="botStore.activeBot.startBot()"
    >
      <template #icon>
        <i-mdi-play />
      </template>
    </Button>
    <Button
      size="large"
      severity="secondary"
      :disabled="!botStore.activeBot.isTrading || !isRunning"
      title="停止交易 - 这也会停止处理现有持仓。"
      @click="handleStopBot()"
    >
      <template #icon>
        <i-mdi-stop />
      </template>
    </Button>
    <Button
      size="large"
      severity="secondary"
      :disabled="!botStore.activeBot.isTrading || !isRunning"
      title="暂停 (停止购买) - Freqtrade 将继续处理现有持仓，但不会建立新的仓位或增加仓位大小。"
      @click="handleStopBuy()"
    >
      <template #icon>
        <i-mdi-pause />
      </template>
    </Button>
    <Button
      size="large"
      severity="secondary"
      :disabled="!botStore.activeBot.isTrading"
      title="重新加载配置 - 重新加载包括策略在内的配置，重置所有动态修改的设置。"
      @click="handleReloadConfig()"
    >
      <template #icon>
        <i-mdi-reload />
      </template>
    </Button>
    <Button
      severity="secondary"
      size="large"
      :disabled="!botStore.activeBot.isTrading"
      title="全部强制平仓"
      @click="handleForceExit()"
    >
      <template #icon>
        <i-mdi-close-box-multiple />
      </template>
    </Button>
    <Button
      v-if="botStore.activeBot.botState && botStore.activeBot.botState.force_entry_enable"
      size="large"
      severity="secondary"
      :disabled="!botStore.activeBot.isTrading || !isRunning"
      title="强制进场 - 立即以可选价格进场交易。出场将根据策略规则处理。"
      @click="forceEnter = true"
    >
      <template #icon>
        <i-mdi-plus-box-multiple-outline />
      </template>
    </Button>
    <Button
      v-if="botStore.activeBot.isWebserverMode && false"
      size="large"
      severity="secondary"
      :disabled="botStore.activeBot.isTrading"
      title="启动交易模式"
      @click="botStore.activeBot.startTrade()"
    >
      <template #icon>
        <i-mdi-play />
      </template>
    </Button>
    <ForceEntryForm v-model="forceEnter" :pair="botStore.activeBot.selectedPair" />
    <MessageBox ref="msgBox" />
  </div>
</template>
