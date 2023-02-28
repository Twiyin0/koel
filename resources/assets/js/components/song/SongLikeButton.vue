<template>
  <button :title="title" type="button" @click.stop="toggleLike">
    <icon v-if="song.liked" :icon="faHeart" />
    <icon v-else :icon="faEmptyHeart" />
  </button>
</template>

<script lang="ts" setup>
import { faHeart } from '@fortawesome/free-solid-svg-icons'
import { faHeart as faEmptyHeart } from '@fortawesome/free-regular-svg-icons'
import { computed, toRefs } from 'vue'
import { favoriteStore } from '@/stores'

const props = defineProps<{ song: Song }>()
const { song } = toRefs(props)

const title = computed(() => `${song.value.liked ? '不再喜欢' : '喜欢'} ${song.value.artist_name} 的 ${song.value.title}`)

const toggleLike = () => favoriteStore.toggleOne(song.value)
</script>
