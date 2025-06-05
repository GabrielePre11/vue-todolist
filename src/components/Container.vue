<script setup lang="ts">
type ContainerProps = {
  class?: string;
};
const props = defineProps<ContainerProps>();
</script>

<template>
  <div :class="['container', props.class]">
    <slot />
  </div>
</template>

<style scoped>
.container {
  padding-inline: 16px;
  max-width: 768px;
  margin-inline: auto;
  margin-block-start: 150px;
  width: 100%;
}
</style>
