<script setup>
defineProps(["click"]);
</script>
<template>
  <div
    v-if="$clicks >= click"
    :class="{
      outline: $clicks === click,
      'outline-blue-500/50': true,
      'outline-4': true,
    }"
  >
    <slot />
  </div>
</template>
