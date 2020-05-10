<template>
  <div class="card" :style="style">
    <div class="card-title">
      {{ title }}
    </div>

    <div class="card-body">
      {{ body }}
    </div>
  </div>
</template>

<script>
import { defineComponent } from 'vue'

export default defineComponent({
  props: {
    title: {
      type: String
    },
    body: {
      type: String
    },
    color: {
      type: String
    }
  },

  setup(props) {
    return {
      style: {
        border: `2px solid ${props.color}`
      }
    }
  }
})
</script>

<style scoped>
.card {
  font-family: 'Courier New', Courier, monospace;
  margin: 10px 0;
  border-radius: 8px;
  box-shadow: 0px 0px 5px 2px lightgrey;
}

.card-title, .card-body {
  padding: 10px;
}
</style>
