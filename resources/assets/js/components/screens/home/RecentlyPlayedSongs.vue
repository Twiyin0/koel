<template>
  <section class="recent">
    <h1>
      最近播放
      <Btn
        v-if="songs.length"
        orange
        rounded
        small
        @click.prevent="goToRecentlyPlayedScreen"
      >
        查看所有
      </Btn>
    </h1>

    <ol v-if="loading" class="recent-song-list">
      <li v-for="i in 3" :key="i">
        <SongCardSkeleton />
      </li>
    </ol>
    <template v-else>
      <ol v-if="songs.length" class="recent-song-list">
        <li v-for="song in songs" :key="song.id">
          <SongCard :song="song" />
        </li>
      </ol>
      <p v-else class="text-secondary">这里妹有歌歌</p>
    </template>
  </section>
</template>

<script lang="ts" setup>
import { toRef, toRefs } from 'vue'
import { overviewStore } from '@/stores'
import { useRouter } from '@/composables'

import Btn from '@/components/ui/Btn.vue'
import SongCard from '@/components/song/SongCard.vue'
import SongCardSkeleton from '@/components/ui/skeletons/SongCardSkeleton.vue'

const { go } = useRouter()

const props = withDefaults(defineProps<{ loading?: boolean }>(), { loading: false })
const { loading } = toRefs(props)

const songs = toRef(overviewStore.state, 'recentlyPlayed')

const goToRecentlyPlayedScreen = () => go('recently-played')
</script>
