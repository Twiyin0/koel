<template>
  <form data-testid="update-profile-form" @submit.prevent="update">
    <div class="form-row">
      <label>
        当前密码
        <input
          v-model="profile.current_password"
          v-koel-focus
          name="current_password"
          placeholder="更新信息的先输密码"
          required
          type="password"
        >
      </label>
    </div>

    <div class="form-row">
      <label>
        名称
        <input id="inputProfileName" v-model="profile.name" name="name" required type="text">
      </label>
    </div>

    <div class="form-row">
      <label>
        邮箱
        <input id="inputProfileEmail" v-model="profile.email" name="email" required type="email">
      </label>
    </div>

    <div class="form-row">
      <label>
        新密码
        <input
          id="inputProfileNewPassword"
          v-model="profile.new_password"
          autocomplete="new-password"
          name="new_password"
          placeholder="不改密码留空就可以哦"
          type="password"
        >
        <span class="password-rules help">
          最少 10 个字符。要字符、数字和符号的组合哦~
        </span>
      </label>
    </div>

    <div class="form-row">
      <Btn class="btn-submit" type="submit">保存</Btn>
      <span v-if="isDemo()" class="demo-notice">
        这些更改，在Demo版本不起效哦
      </span>
    </div>
  </form>
</template>

<script lang="ts" setup>
import { onMounted, ref } from 'vue'
import { UpdateCurrentProfileData, userStore } from '@/stores'
import { isDemo, logger, parseValidationError } from '@/utils'
import { useDialogBox, useMessageToaster } from '@/composables'

import Btn from '@/components/ui/Btn.vue'

const { toastSuccess } = useMessageToaster()
const { showErrorDialog } = useDialogBox()
const profile = ref<UpdateCurrentProfileData>({} as unknown as UpdateCurrentProfileData)

onMounted(() => {
  profile.value = {
    name: userStore.current.name,
    email: userStore.current.email,
    current_password: null
  }
})

const update = async () => {
  if (!profile.value) {
    throw Error()
  }

  if (isDemo()) {
    toastSuccess('Profile updated.')
    return
  }

  try {
    await userStore.updateProfile(profile.value)
    profile.value.current_password = null
    delete profile.value.new_password
    toastSuccess('Profile updated.')
  } catch (error: any) {
    const msg = error.response.status === 422 ? parseValidationError(error.response.data)[0] : 'Unknown error.'
    showErrorDialog(msg, 'Error')
    logger.log(error)
  }
}
</script>

<style lang="scss" scoped>
input {
  &[type="text"], &[type="email"], &[type="password"] {
    width: 33%;
  }
}

.password-rules {
  display: block;
  margin-top: .75rem;
}

.demo-notice {
  font-size: .95rem;
  opacity: .7;
  margin-left: 5px;
}

@media only screen and (max-width: 667px) {
  input {
    &[type="text"], &[type="email"], &[type="password"] {
      width: 100%;
      height: 32px;
    }
  }
}
</style>
