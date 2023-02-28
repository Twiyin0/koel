<template>
  <FormBase>
    <form @submit.prevent="submit" @keydown.esc="maybeClose">
      <header>
        <h1>新建智能歌单</h1>
      </header>

      <main>
        <div class="form-row cols">
          <label class="name">
            名称
            <input v-model="name" v-koel-focus name="name" placeholder="歌单名称" required type="text">
          </label>
          <label class="folder">
            收藏夹
            <select v-model="folderId">
              <option :value="null" />
              <option v-for="folder in folders" :key="folder.id" :value="folder.id">{{ folder.name }}</option>
            </select>
          </label>
        </div>

        <div class="form-row rules">
          <RuleGroup
            v-for="(group, index) in collectedRuleGroups"
            :key="group.id"
            :group="group"
            :is-first-group="index === 0"
            @input="onGroupChanged"
          />
          <Btn class="btn-add-group" green small title="Add a new group" uppercase @click.prevent="addGroup">
            <icon :icon="faPlus" />
            组别
          </Btn>
        </div>
      </main>

      <footer>
        <Btn type="submit">保存^_^</Btn>
        <Btn class="btn-cancel" white @click.prevent="maybeClose">取消awa</Btn>
      </footer>
    </form>
  </FormBase>
</template>

<script lang="ts" setup>
import { faPlus } from '@fortawesome/free-solid-svg-icons'
import { ref, toRef } from 'vue'
import { playlistFolderStore, playlistStore } from '@/stores'
import { logger } from '@/utils'
import { useDialogBox, useMessageToaster, useModal, useOverlay, useRouter, useSmartPlaylistForm } from '@/composables'

const {
  Btn,
  FormBase,
  RuleGroup,
  collectedRuleGroups,
  addGroup,
  onGroupChanged
} = useSmartPlaylistForm()

const { showOverlay, hideOverlay } = useOverlay()
const { toastSuccess } = useMessageToaster()
const { showConfirmDialog, showErrorDialog } = useDialogBox()
const { go } = useRouter()
const targetFolder = useModal().getFromContext<PlaylistFolder | null>('folder')

const name = ref('')
const folderId = ref(targetFolder?.id)
const folders = toRef(playlistFolderStore.state, 'folders')

const emit = defineEmits<{ (e: 'close'): void }>()
const close = () => emit('close')

const isPristine = () => name.value === ''
  && folderId.value === targetFolder?.id
  && collectedRuleGroups.value.length === 0

const maybeClose = async () => {
  if (isPristine()) {
    close()
    return
  }

  await showConfirmDialog('Discard all changes?') && close()
}

const submit = async () => {
  showOverlay()

  try {
    const playlist = await playlistStore.store(name.value, {
      rules: collectedRuleGroups.value,
      folder_id: folderId.value
    })

    close()
    toastSuccess(`Playlist "${playlist.name}" created.`)
    go(`playlist/${playlist.id}`)
  } catch (error) {
    showErrorDialog('Something went wrong. Please try again.', 'Error')
    logger.error(error)
  } finally {
    hideOverlay()
  }
}
</script>
