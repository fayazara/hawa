<template>
  <span :class="name" />
</template>

<script setup>
defineProps({
  name: String,
  required: true,
});
</script>
