<template>
  <article id="lyrics">
    <div class="content">
      <template v-if="song">
        <div v-show="song.lyrics">
          <pre ref="lyricsContainer">{{ lyrics }}</pre>
          <Magnifier class="magnifier" @in="zoomLevel++" @out="zoomLevel--" />
        </div>
        <p v-if="song.id && !song.lyrics" class="none text-secondary">
          <template v-if="isAdmin">
            妹有歌词欸……可以
            <button class="text-highlight" type="button" @click.prevent="showEditSongForm">
              点这里
            </button>
            来加上点调皮的歌词哦
          </template>
          <span v-else>你这歌词是纯音乐做的罢&lt;( ￣^￣)</span>
        </p>
      </template>
    </div>
  </article>
</template>

<script lang="ts" setup>
import { computed, defineAsyncComponent, onMounted, ref, toRefs, watch } from 'vue'
import { eventBus } from '@/utils'
import { useAuthorization } from '@/composables'
import { preferenceStore as preferences } from '@/stores'

const Magnifier = defineAsyncComponent(() => import('@/components/ui/Magnifier.vue'))

const props = defineProps<{ song: Song }>()
const { song } = toRefs(props)

const { isAdmin } = useAuthorization()

const lyricsContainer = ref<HTMLElement>()
const zoomLevel = ref(preferences.lyricsZoomLevel || 1)

const showEditSongForm = () => eventBus.emit('MODAL_SHOW_EDIT_SONG_FORM', song.value, 'lyrics')

const lyrics = computed(() => {
  // This trick converts CRs (\r) to LF (\n) using JavaScript's implicit DOM-writing behavior
  const div = document.createElement('div')
  div.innerHTML = song.value.lyrics
  return div.innerHTML
})

const setFontSize = () => {
  if (lyricsContainer.value) {
    lyricsContainer.value.style.fontSize = `${1 + (zoomLevel.value - 1) * 0.2}rem`
  }
}

watch(zoomLevel, level => {
  setFontSize()
  preferences.lyricsZoomLevel = level
})

onMounted(() => setFontSize())
</script>

<style lang="scss" scoped>
.content {
  position: relative;

  .magnifier {
    opacity: 0;
    position: absolute;
    top: 0;
    right: 0;

    @media (hover: none) {
      opacity: 1;
    }
  }

  &:hover .magnifier {
    opacity: .5;

    &:hover {
      opacity: 1;
    }
  }
}

pre {
  white-space: pre-wrap;
  line-height: 1.7;
}
</style>
