<template>
  <div>
    <h1>Stories</h1>

    <div v-for="story in stories" :key="story">
      <RouterLink :to="`/${story}`">
        Stories of {{ story }}
      </RouterLink>
    </div>

    <div v-if="currentStory">
      <h2>Stories for {{ currentStory.name }}</h2>
      <component :is="currentStory.component" />
    </div>
  </div>
</template>

<script>
import { useRoute } from 'vue-router'
import { computed } from 'vue'
import ButtonStories from './Button.stories.vue'
import CardStories from './Card.stories.vue'

export default {
  setup() {
    const storyMap = {
      button: {
        name: 'button',
        component: ButtonStories
      },
      card: {
        name: 'card',
        component: CardStories
      }
    }
    const stories = Object.keys(storyMap)
    const route = useRoute()

    const currentStory = computed(() => {
      const story = storyMap[route.params.name]
      if (story) {
        return story
      }
    })

    return {
      storyMap,
      stories,
      currentStory,
    }
  },
}
</script>
