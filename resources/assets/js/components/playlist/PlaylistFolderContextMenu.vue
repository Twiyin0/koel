<template>
  <ContextMenuBase ref="base">
    <template v-if="folder">
      <template v-if="playable">
        <li @click="play">播放所有</li>
        <li @click="shuffle">随机播放</li>
        <li class="separator" />
      </template>
      <li @click="createPlaylist">新建歌单</li>
      <li @click="createSmartPlaylist">新建智能歌单</li>
      <li class="separator" />
      <li @click="rename">重命名</li>
      <li @click="destroy">删掉惹！</li>
    </template>
  </ContextMenuBase>
</template>

<script lang="ts" setup>
import { computed, ref } from 'vue'
import { eventBus } from '@/utils'
import { playlistStore, songStore } from '@/stores'
import { playbackService } from '@/services'
import { useContextMenu, useMessageToaster, useRouter } from '@/composables'

const { base, ContextMenuBase, open, trigger } = useContextMenu()
const { go } = useRouter()
const { toastWarning } = useMessageToaster()

const folder = ref<PlaylistFolder>()

const playlistsInFolder = computed(() => folder.value ? playlistStore.byFolder(folder.value) : [])
const playable = computed(() => playlistsInFolder.value.length > 0)

const play = () => trigger(async () => {
  const songs = await songStore.fetchForPlaylistFolder(folder.value!)

  if (songs.length) {
    playbackService.queueAndPlay(songs)
    go('queue')
  } else {
    toastWarning('No songs available.')
  }
})

const shuffle = () => trigger(async () => {
  const songs = await songStore.fetchForPlaylistFolder(folder.value!)

  if (songs.length) {
    playbackService.queueAndPlay(songs, true)
    go('queue')
  } else {
    toastWarning('No songs available.')
  }
})

const createPlaylist = () => trigger(() => eventBus.emit('MODAL_SHOW_CREATE_PLAYLIST_FORM', folder.value!))
const createSmartPlaylist = () => trigger(() => eventBus.emit('MODAL_SHOW_CREATE_SMART_PLAYLIST_FORM', folder.value!))
const rename = () => trigger(() => eventBus.emit('MODAL_SHOW_EDIT_PLAYLIST_FOLDER_FORM', folder.value!))
const destroy = () => trigger(() => eventBus.emit('PLAYLIST_FOLDER_DELETE', folder.value!))

eventBus.on('PLAYLIST_FOLDER_CONTEXT_MENU_REQUESTED', async (e, _folder) => {
  folder.value = _folder
  await open(e.pageY, e.pageX)
})
</script>
