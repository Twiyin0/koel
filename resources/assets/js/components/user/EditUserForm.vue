<template>
  <form data-testid="edit-user-form" @submit.prevent="submit" @keydown.esc="maybeClose">
    <header>
      <h1>编辑小伙伴信息</h1>
    </header>

    <main>
      <div class="form-row">
        <label>
          名称
          <input v-model="updateData.name" v-koel-focus name="name" required title="Name" type="text">
        </label>
      </div>
      <div class="form-row">
        <label>
          邮箱
          <input v-model="updateData.email" name="email" required title="Email" type="email">
        </label>
      </div>
      <div class="form-row">
        <label>
          密码
          <input
            v-model="updateData.password"
            autocomplete="new-password"
            name="password"
            placeholder="Leave blank for no changes"
            type="password"
          >
        </label>
        <p class="help">最少 10 个字符。要字符、数字和符号的组合哦</p>
      </div>
      <div class="form-row">
        <label>
          <CheckBox v-model="updateData.is_admin" name="is_admin" />
          设为小管管（管理）
          <TooltipIcon title="管理员可以执行管理任务，比如管理用户和上传歌曲" />
        </label>
      </div>
    </main>

    <footer>
      <Btn class="btn-update" type="submit">Update</Btn>
      <Btn class="btn-cancel" white @click.prevent="maybeClose">Cancel</Btn>
    </footer>
  </form>
</template>

<script lang="ts" setup>
import { isEqual } from 'lodash'
import { reactive, watch } from 'vue'
import { logger, parseValidationError } from '@/utils'
import { UpdateUserData, userStore } from '@/stores'
import { useDialogBox, useMessageToaster, useModal, useOverlay } from '@/composables'

import Btn from '@/components/ui/Btn.vue'
import TooltipIcon from '@/components/ui/TooltipIcon.vue'
import CheckBox from '@/components/ui/CheckBox.vue'

const { showOverlay, hideOverlay } = useOverlay()
const { toastSuccess } = useMessageToaster()
const { showConfirmDialog, showErrorDialog } = useDialogBox()
const user = useModal().getFromContext<User>('user')

let originalData: UpdateUserData
let updateData: UpdateUserData

watch(user, () => {
  originalData = {
    name: user.name,
    email: user.email,
    is_admin: user.is_admin
  }

  updateData = reactive(Object.assign({}, originalData))
}, { immediate: true })

const submit = async () => {
  showOverlay()

  try {
    await userStore.update(user, updateData)
    toastSuccess('User profile updated.')
    close()
  } catch (error: any) {
    const msg = error.response.status === 422 ? parseValidationError(error.response.data)[0] : 'Unknown error.'
    showErrorDialog(msg, 'Error')
    logger.error(error)
  } finally {
    hideOverlay()
  }
}

const emit = defineEmits<{ (e: 'close'): void }>()
const close = () => emit('close')

const maybeClose = async () => {
  if (isEqual(originalData, updateData)) {
    close()
    return
  }

  await showConfirmDialog('Discard all changes?') && close()
}
</script>

<style lang="scss" scoped>
.help {
  margin-top: .75rem;
}
</style>
