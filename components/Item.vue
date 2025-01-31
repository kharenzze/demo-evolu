<script setup lang="ts">
const props = defineProps<{
  item: any;
}>();
</script>

<template>
  <div>
    <pre>{{ item }}</pre>
  </div>
</template>
