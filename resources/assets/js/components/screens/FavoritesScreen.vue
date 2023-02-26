<template>
  <section id="favoritesWrapper">
    <ScreenHeader :layout="songs.length === 0 ? 'collapsed' : headerLayout">
      喜欢的歌哦
      <ControlsToggle v-model="showingControls" />

      <template #thumbnail>
        <ThumbnailStack :thumbnails="thumbnails" />
      </template>

      <template v-if="songs.length" #meta>
        <span>{{ pluralize(songs, 'song') }}</span>
        <span>{{ duration }}</span>

        <a
          v-if="allowDownload"
          class="download"
          role="button"
          title="下载歌单里所有歌曲"
          @click.prevent="download"
        >
          下载所有
        </a>
      </template>

      <template #controls>
        <SongListControls
          v-if="songs.length && (!isPhone || showingControls)"
          @filter="applyFilter"
          @play-all="playAll"
          @play-selected="playSelected"
        />
      </template>
    </ScreenHeader>

    <SongListSkeleton v-if="loading" />
    <SongList
      v-if="songs.length"
      ref="songList"
      @sort="sort"
      @press:delete="removeSelected"
      @press:enter="onPressEnter"
      @scroll-breakpoint="onScrollBreakpoint"
    />

    <ScreenEmptyState v-else>
      <template #icon>
        <icon :icon="faHeartBroken" />
      </template>
      妹有喜欢的歌欸……
      <span class="secondary d-block">
        点击&nbsp;
        <icon :icon="faHeart" />&nbsp;
        图标添加喜欢的歌~
      </span>
    </ScreenEmptyState>
  </section>
</template>

<script lang="ts" setup>
import { faHeartBroken } from '@fortawesome/free-solid-svg-icons'
import { faHeart } from '@fortawesome/free-regular-svg-icons'
import { pluralize } from '@/utils'
import { commonStore, favoriteStore } from '@/stores'
import { downloadService } from '@/services'
import { useRouter, useSongList } from '@/composables'
import { nextTick, ref, toRef } from 'vue'

import ScreenHeader from '@/components/ui/ScreenHeader.vue'
import ScreenEmptyState from '@/components/ui/ScreenEmptyState.vue'
import SongListSkeleton from '@/components/ui/skeletons/SongListSkeleton.vue'

const {
  SongList,
  SongListControls,
  ControlsToggle,
  ThumbnailStack,
  headerLayout,
  songs,
  songList,
  duration,
  thumbnails,
  selectedSongs,
  showingControls,
  isPhone,
  onPressEnter,
  playAll,
  playSelected,
  applyFilter,
  onScrollBreakpoint,
  sort
} = useSongList(toRef(favoriteStore.state, 'songs'))

const allowDownload = toRef(commonStore.state, 'allow_download')

const download = () => downloadService.fromFavorites()
const removeSelected = () => selectedSongs.value.length && favoriteStore.unlike(selectedSongs.value)

let initialized = false
const loading = ref(false)

const fetchSongs = async () => {
  loading.value = true
  await favoriteStore.fetch()
  loading.value = false
  await nextTick()
  sort()
}

useRouter().onScreenActivated('Favorites', async () => {
  if (!initialized) {
    initialized = true
    await fetchSongs()
  }
})
</script>
