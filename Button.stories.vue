<template>
  <Button v-for="color in colors" :key="color" :background="color">Button</Button>
</template>

<script>
import Button from '../Button.vue'

export default {
  components: {
    Button
  },

  data() {
    return {
      colors: ['purple', 'red', 'blue', 'green', 'pink']
    }
  }
}
</script>
