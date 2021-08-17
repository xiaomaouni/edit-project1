<template>
  <div>
    文本
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
  //   name: 'Txt',
  //   props: ['data'],
  //   data() {
  //     return {}
  //   },
  //   mounted() {}
  // }
</script>

<style>

</style>