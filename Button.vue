<template>
  <button class="button" :style="style">
    <slot />
  </button>
</template>

<script>
import { defineComponent } from 'vue'

export default {
  props: ['background', 'color'],

  computed: {
    style() {
      return {
        background: this.background,
        color: this.color
      }
    }
  }
}
</script>

<style scoped>
.button {
  height: 40px;
  border-radius: 5px;
  font-size: 1.1rem;
  color: white;
  box-shadow: 0px 0px 10px 1px grey;
  margin: 5px;
}
</style>
