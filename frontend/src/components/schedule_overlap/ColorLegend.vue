<template>
  <div class="tw-flex tw-flex-col tw-gap-1">
    <div class="tw-text-sm tw-font-medium">種類</div>
    <div class="tw-flex tw-items-center tw-gap-2">
      <div
        class="tw-h-4 tw-w-4 tw-rounded tw-border tw-border-gray tw-bg-[#00994C77]"
      ></div>
      <span class="tw-text-sm">有空</span>
    </div>
    <div class="tw-flex tw-items-center tw-gap-2">
      <div
        class="tw-h-4 tw-w-4 tw-rounded tw-border tw-border-gray tw-bg-yellow"
      ></div>
      <span class="tw-text-sm">如果有需要</span>
    </div>
    <div class="tw-flex tw-items-center tw-gap-2">
      <div
        class="tw-h-4 tw-w-4 tw-rounded tw-border tw-border-gray tw-bg-[#E523232D]"
      ></div>
      <span class="tw-text-sm">沒空</span>
    </div>
  </div>
</template>

<script>
export default {
  name: "ColorLegend",
}
</script>
