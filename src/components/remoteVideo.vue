<template>
  <video class="remote-video"></video>
</template>

<script>
export default {
  props: ['remoteVideo'],
  mounted() {
    this.$el.srcObject = this.remoteVideo.getMediaStream();
  }
}
</script>

<style scoped>
video {
  width: 100%;
}
</style>
