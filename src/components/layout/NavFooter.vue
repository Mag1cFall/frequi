<script setup lang="ts">
const botStore = useBotStore();
</script>

<template>
  <footer class="md:hidden">
    <!-- Only visible on xs (phone) viewport! -->
    <hr class="my-0" />
    <div class="flex gap-2 justify-between px-2">
      <Button
        v-if="!botStore.canRunBacktest"
        icon-pos="top"
        variant="link"
        size="small"
        as="router-link"
        class="align-items-center"
        to="/open_trades"
        label="交易"
      >
        <template #icon>
          <i-mdi-folder-open height="24" width="24" />
        </template>
      </Button>
      <Button
        v-if="!botStore.canRunBacktest"
        icon-pos="top"
        variant="link"
        size="small"
        as="router-link"
        class="align-items-center"
        to="/trade_history"
        label="歷史紀錄"
      >
        <template #icon>
          <i-mdi-folder-lock height="24" width="24" />
        </template>
      </Button>
      <Button
        v-if="!botStore.canRunBacktest"
        icon-pos="top"
        variant="link"
        size="small"
        as="router-link"
        class="align-items-center"
        to="/pairlist"
        label="交易對列表"
      >
        <template #icon>
          <i-mdi-view-list height="24" width="24" />
        </template>
      </Button>
      <Button
        v-if="!botStore.canRunBacktest"
        icon-pos="top"
        variant="link"
        size="small"
        as="router-link"
        class="align-items-center"
        to="/balance"
        label="餘額"
      >
        <template #icon>
          <i-mdi-bank height="24" width="24" />
        </template>
      </Button>
      <Button
        v-if="!botStore.canRunBacktest"
        icon-pos="top"
        variant="link"
        size="small"
        as="router-link"
        class="align-items-center"
        to="/dashboard"
        label="儀表板"
      >
        <template #icon>
          <i-mdi-view-dashboard-outline height="24" width="24" />
        </template>
      </Button>
    </div>
  </footer>
</template>
