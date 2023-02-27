<template>
  <div>
    <div v-if="!isPhone" class="form-row">
      <label>
        <CheckBox v-model="preferences.notify" name="notify" />
        显示正在播放的通知
      </label>
    </div>
    <div v-if="!isPhone" class="form-row">
      <label>
        <CheckBox v-model="preferences.confirmClosing" name="confirm_closing" />
        在关闭钱确认
      </label>
    </div>
    <div v-if="isPhone" class="form-row">
      <label>
        <CheckBox v-model="preferences.transcodeOnMobile" name="transcode_on_mobile" />
        在移动设备上以 128kbps 的速度转换和播放音乐
      </label>
    </div>
    <div class="form-row">
      <label>
        <CheckBox v-model="preferences.showAlbumArtOverlay" name="show_album_art_overlay" />
        显示当前专辑封面的半透明模糊叠加
      </label>
    </div>
  </div>
</template>

<script lang="ts" setup>
import isMobile from 'ismobilejs'
import { preferenceStore as preferences } from '@/stores'
import CheckBox from '@/components/ui/CheckBox.vue'

const isPhone = isMobile.phone
</script>

<style lang="scss" scoped>
label {
  font-size: 1rem;
}
</style>
