<template>
  <video ref="myVideoEl" :srcObject="myStreamSrc" autoplay="autoplay" />
</template>
<script>
export default {
  name: 'VideoComponent',
  props: ['myStreamSrc']
}
</script>
<style lang="scss" scoped>

</style>
