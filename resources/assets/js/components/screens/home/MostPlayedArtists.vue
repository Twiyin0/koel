<template>
  <section>
    <h1>Top 艺术家</h1>

    <ol v-if="loading" class="two-cols top-album-list">
      <li v-for="i in 4" :key="i">
        <ArtistCardSkeleton layout="compact" />
      </li>
    </ol>
    <template v-else>
      <ol v-if="artists.length" class="two-cols top-artist-list">
        <li v-for="artist in artists" :key="artist.id">
          <ArtistCard :artist="artist" layout="compact" />
        </li>
      </ol>
      <p v-else class="text-secondary">这里妹有艺术家的位置啦！！</p>
    </template>
  </section>
</template>

<script lang="ts" setup>
import { toRef, toRefs } from 'vue'
import { overviewStore } from '@/stores'
import ArtistCard from '@/components/artist/ArtistCard.vue'
import ArtistCardSkeleton from '@/components/ui/skeletons/ArtistAlbumCardSkeleton.vue'

const props = withDefaults(defineProps<{ loading?: boolean }>(), { loading: false })
const { loading } = toRefs(props)

const artists = toRef(overviewStore.state, 'mostPlayedArtists')
</script>
