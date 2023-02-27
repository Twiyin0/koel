<template>
  <ContextMenuBase ref="base" data-testid="song-context-menu" extra-class="song-menu">
    <template v-if="onlyOneSongSelected">
      <li @click.stop.prevent="doPlayback">
        <span v-if="firstSongPlaying">暂停</span>
        <span v-else>播放</span>
      </li>
      <li @click="viewAlbumDetails(songs[0].album_id)">哪个专辑里的捏</li>
      <li @click="viewArtistDetails(songs[0].artist_id)">是谁唱的捏~</li>
    </template>
    <li class="has-sub">
      添加到
      <ul class="menu submenu menu-add-to">
        <template v-if="queue.length">
          <li v-if="currentSong" @click="queueSongsAfterCurrent">放到下一曲播放</li>
          <li @click="queueSongsToBottom">当前播放最后</li>
          <li @click="queueSongsToTop">当前播放最前</li>
        </template>
        <li v-else @click="queueSongsToBottom">当前播放</li>
        <template v-if="!isFavoritesScreen">
          <li class="separator" />
          <li @click="addSongsToFavorite">喜欢的歌曲</li>
        </template>
        <li v-if="normalPlaylists.length" class="separator" />
        <ul v-if="normalPlaylists.length" v-koel-overflow-fade class="playlists">
          <li v-for="p in normalPlaylists" :key="p.id" @click="addSongsToExistingPlaylist(p)">{{ p.name }}</li>
        </ul>
        <li class="separator" />
        <li @click="addSongsToNewPlaylist">新建歌单</li>
      </ul>
    </li>

    <template v-if="isQueueScreen">
      <li class="separator" />
      <li @click="removeFromQueue">从当前播放移除</li>
      <li class="separator" />
    </template>

    <template v-if="isFavoritesScreen">
      <li class="separator" />
      <li @click="removeFromFavorites">爱是会转移的……</li>
      <li class="separator" />
    </template>

    <li v-if="isAdmin" @click="openEditForm">编辑</li>
    <li v-if="allowDownload" @click="download">下载</li>
    <li v-if="onlyOneSongSelected" @click="copyUrl">分享</li>

    <template v-if="canBeRemovedFromPlaylist">
      <li class="separator" />
      <li @click="removeFromPlaylist">这里妹有你的位置啦！</li>
    </template>

    <template v-if="isAdmin">
      <li class="separator" />
      <li @click="deleteFromFilesystem">从文件中删除</li>
    </template>
  </ContextMenuBase>
</template>

<script lang="ts" setup>
import { computed, ref, toRef } from 'vue'
import { arrayify, copyText, eventBus, pluralize } from '@/utils'
import { commonStore, favoriteStore, playlistStore, queueStore, songStore, userStore } from '@/stores'
import { downloadService, playbackService } from '@/services'
import {
  useAuthorization,
  useContextMenu,
  useDialogBox,
  useMessageToaster,
  usePlaylistManagement,
  useRouter,
  useSongMenuMethods
} from '@/composables'

const { toastSuccess } = useMessageToaster()
const { showConfirmDialog } = useDialogBox()
const { go, getRouteParam, isCurrentScreen } = useRouter()
const { isAdmin } = useAuthorization()
const { base, ContextMenuBase, open, close, trigger } = useContextMenu()
const { removeSongsFromPlaylist } = usePlaylistManagement()

const songs = ref<Song[]>([])

const {
  queueSongsAfterCurrent,
  queueSongsToBottom,
  queueSongsToTop,
  addSongsToFavorite,
  addSongsToExistingPlaylist,
  addSongsToNewPlaylist
} = useSongMenuMethods(songs, close)

const playlists = toRef(playlistStore.state, 'playlists')
const allowDownload = toRef(commonStore.state, 'allow_download')
const queue = toRef(queueStore.state, 'songs')
const currentSong = toRef(queueStore, 'current')

const onlyOneSongSelected = computed(() => songs.value.length === 1)
const firstSongPlaying = computed(() => songs.value.length ? songs.value[0].playback_state === 'Playing' : false)
const normalPlaylists = computed(() => playlists.value.filter(playlist => !playlist.is_smart))

const canBeRemovedFromPlaylist = computed(() => {
  if (!isCurrentScreen('Playlist')) return false
  const playlist = playlistStore.byId(parseInt(getRouteParam('id')!))
  return playlist && !playlist.is_smart
})

const isQueueScreen = computed(() => isCurrentScreen('Queue'))
const isFavoritesScreen = computed(() => isCurrentScreen('Favorites'))

const doPlayback = () => trigger(() => {
  if (!songs.value.length) return

  switch (songs.value[0].playback_state) {
    case 'Playing':
      playbackService.pause()
      break

    case 'Paused':
      playbackService.resume()
      break

    default:
      queueStore.queueIfNotQueued(songs.value[0])
      playbackService.play(songs.value[0])
      break
  }
})

const openEditForm = () => trigger(() => songs.value.length && eventBus.emit('MODAL_SHOW_EDIT_SONG_FORM', songs.value))
const viewAlbumDetails = (albumId: number) => trigger(() => go(`album/${albumId}`))
const viewArtistDetails = (artistId: number) => trigger(() => go(`artist/${artistId}`))
const download = () => trigger(() => downloadService.fromSongs(songs.value))

const removeFromPlaylist = () => trigger(async () => {
  const playlist = playlistStore.byId(parseInt(getRouteParam('id')!))
  if (!playlist) return

  await removeSongsFromPlaylist(playlist, songs.value)
})

const removeFromQueue = () => trigger(() => queueStore.unqueue(songs.value))
const removeFromFavorites = () => trigger(() => favoriteStore.unlike(songs.value))

const copyUrl = () => trigger(() => {
  copyText(songStore.getShareableUrl(songs.value[0]))
  toastSuccess('URL copied to clipboard.')
})

const deleteFromFilesystem = () => trigger(async () => {
  if (await showConfirmDialog('Delete selected song(s) from the filesystem? This action is NOT reversible!')) {
    await songStore.deleteFromFilesystem(songs.value)
    toastSuccess(`Deleted ${pluralize(songs.value, 'song')} from the filesystem.`)
    eventBus.emit('SONGS_DELETED', songs.value)
  }
})

eventBus.on('SONG_CONTEXT_MENU_REQUESTED', async (e, _songs) => {
  songs.value = arrayify(_songs)
  await open(e.pageY, e.pageX)
})
</script>

<style lang="scss" scoped>
ul.playlists {
  position: relative;
  max-height: 192px;
  overflow-y: auto;
}
</style>
