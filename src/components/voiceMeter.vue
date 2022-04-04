<template>
  <div class="containerStyle">
    <div class="barStyle" :style="{height: level2 + 'px'}"></div>
    <div class="barStyle" :style="{height: level1 + 'px'}"></div>
    <div class="barStyle" :style="{height: level2 + 'px'}"></div>
  </div>
</template>

<script>
export default {
  props: ['level'],
  data() {
    return {
      level1: 0,
      level2: 0
    };
  },
  watch: {
    level() {
      const edited = this.level > 10 ? this.level : 0;
      this.level1 = this.scaleUp(edited, 100, 30);
      this.level2 = this.scaleUp(edited, 100, 20);
    }
  },
  methods: {
    scaleUp(value, total, max) {
      const val = (value / total * max);
      return val < 2 ? 2 : val;
    }
  }
}
</script>

<style scoped>
.containerStyle {
  width: 28px;
  height: 28px;
  background-color: yellow;
  border-radius: 50%;
  display: flex;
  justify-content: center;
  align-items: center;
  color: black;
  position: absolute;
  left: 5px;
  top: 5px;
}

.barStyle {
  width: 3px;
  border-radius: 3px;
  background-color: black;
  transition: height 0.1s linear;
  will-change: height;
  margin-left: 1px;
}
</style>
