<template>
  <div class="Button">
    <el-button disabled>默认按钮</el-button>
  </div>
</template>

<script setup>
  import {
    defineProps,
    reactive,
    ref,
    getCurrentInstance,
    onMounted,
    inject,
  } from 'vue';
  let {
    proxy
  } = getCurrentInstance();
  let props = defineProps({
    data: Object,
    type: {
      type: Number,
      default: 0
    }
  });
  // export default {
  //   name: 'Button',
  //   props: ['data']
  // }
</script>

<style>

</style>