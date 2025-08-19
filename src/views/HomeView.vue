<template>
  <div class="mt-5">
    <div class="flex justify-center">
      <BotList class="max-w-xl" />
    </div>
    <Divider />
    <div
      title="Freqtrade logo"
      class="logo-svg my-5 mx-auto dark:bg-white bg-black sm:w-[250px] sm:h-[250px] w-[150px] h-[150px] transition-all duration-300"
    />
    <h1 class="font-bold text-2xl sm:text-4xl mb-4 transition-all">歡迎使用 Freqtrade UI</h1>
    <div>此頁面可讓您控制您的交易機器人。</div>
    <Divider />
    <p class="mb-2">
      如果您需要任何幫助，請參閱
      <a
        class="text-primary underline cursor-pointer"
        href="https://www.freqtrade.io/en/latest/"
        target="_blank"
      >
        Freqtrade 文件
      </a>
      。
    </p>

    <p class="mb-5">祝您使用愉快 - <i>Freqtrade 團隊敬上</i></p>
  </div>
</template>

<style lang="css" scoped>
.logo-svg {
  -webkit-mask: url(../assets/freqtrade-logo-mask.png) no-repeat center;
  -webkit-mask-size: 240px 240px;
  mask: url(../assets/freqtrade-logo-mask.png) no-repeat center;
  mask-size: contain;
}
</style>
