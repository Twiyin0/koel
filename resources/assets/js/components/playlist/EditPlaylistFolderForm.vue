<template>
  <form @submit.prevent="submit" @keydown.esc="maybeClose">
    <header>
      <h1>重命名歌单</h1>
    </header>

    <main>
      <div class="form-row">
        <input
          v-model="name"
          v-koel-focus
          name="name"
          placeholder="歌单名称"
          required
          title="歌单名称"
          type="text"
        >
      </div>
    </main>

    <footer>
      <Btn type="submit">保存</Btn>
      <Btn white @click.prevent="maybeClose">取消</Btn>
    </footer>
  </form>
</template>

<script lang="ts" setup>
import { ref } from 'vue'
import { logger } from '@/utils'
import { playlistFolderStore } from '@/stores'
import { useDialogBox, useMessageToaster, useModal, useOverlay } from '@/composables'

import Btn from '@/components/ui/Btn.vue'

const { showOverlay, hideOverlay } = useOverlay()
const { toastSuccess } = useMessageToaster()
const { showConfirmDialog, showErrorDialog } = useDialogBox()
const folder = useModal().getFromContext<PlaylistFolder>('folder')

const name = ref(folder.name)

const submit = async () => {
  showOverlay()

  try {
    await playlistFolderStore.rename(folder, name.value)
    toastSuccess('Playlist folder renamed.')
    close()
  } catch (error) {
    showErrorDialog('Something went wrong. Please try again.', 'Error')
    logger.error(error)
  } finally {
    hideOverlay()
  }
}

const emit = defineEmits<{ (e: 'close'): void }>()
const close = () => emit('close')

const maybeClose = async () => {
  if (name.value.trim() === folder.name) {
    close()
    return
  }

  await showConfirmDialog('Discard all changes?') && close()
}
</script>
