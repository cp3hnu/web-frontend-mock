<template>
  <div class="slidev-layout default">
    <slot />
    <div grid="~ cols-2 gap-4">
      <div><slot name="left" /></div>
      <div><slot name="right" /></div>
    </div>
  </div>
</template>
<style></style>
