<template>
  <section id="settingsWrapper">
    <ScreenHeader>设置</ScreenHeader>

    <form class="main-scroll-wrap" @submit.prevent="confirmThenSave">
      <div class="form-row">
        <label for="inputSettingsPath">音乐库路径</label>

        <p id="mediaPathHelp" class="help">
          包含媒体的服务器目录的<em>绝对</em>路径。
          Koel 将扫描此目录中的歌曲并提取任何可用信息。<br>
          扫描可能需要一段时间，特别是如果您有很多歌曲，请耐心等待。
        </p>

        <input
          id="inputSettingsPath"
          v-model="mediaPath"
          aria-describedby="mediaPathHelp"
          name="media_path"
          type="text"
        >
      </div>

      <div class="form-row">
        <Btn type="submit">扫描</Btn>
      </div>
    </form>
  </section>
</template>

<script lang="ts" setup>
import { computed, ref } from 'vue'
import { settingStore } from '@/stores'
import { forceReloadWindow, parseValidationError } from '@/utils'
import { useDialogBox, useMessageToaster, useOverlay, useRouter } from '@/composables'

import ScreenHeader from '@/components/ui/ScreenHeader.vue'
import Btn from '@/components/ui/Btn.vue'

const { toastSuccess } = useMessageToaster()
const { showConfirmDialog, showErrorDialog } = useDialogBox()
const { go } = useRouter()
const { showOverlay, hideOverlay } = useOverlay()

const mediaPath = ref(settingStore.state.media_path)
const originalMediaPath = mediaPath.value

const shouldWarn = computed(() => {
  // Warn the user if the media path is not empty and about to change.
  if (!originalMediaPath || !mediaPath.value) {
    return false
  }

  return originalMediaPath !== mediaPath.value.trim()
})

const save = async () => {
  showOverlay({ message: 'Scanning…' })

  try {
    await settingStore.update({ media_path: mediaPath.value })
    toastSuccess('Settings saved.')
    // Make sure we're back to home first.
    go('home')
    forceReloadWindow()
  } catch (err: any) {
    const msg = err.response.status === 422 ? parseValidationError(err.response.data)[0] : 'Unknown error.'
    showErrorDialog(msg, 'Error')
  } finally {
    hideOverlay()
  }
}

const confirmThenSave = async () => {
  if (shouldWarn.value) {
    await showConfirmDialog('Changing the media path will essentially remove all existing data – songs, artists, \
          albums, favorites, everything – and empty your playlists! Sure you want to proceed?', 'Confirm')
    && await save()
  } else {
    await save()
  }
}
</script>

<style lang="scss">
#settingsWrapper {
  input[type="text"] {
    width: 50%;
    margin-top: 1rem;
  }

  @media only screen and (max-width: 667px) {
    input[type="text"] {
      width: 100%;
    }
  }
}
</style>
