<template>
  <section>
    <ScreenHeader layout="collapsed">找不到捏</ScreenHeader>

    <div class="main-scroll-wrap">
      <ScreenEmptyState>
        <template #icon>
          <icon :icon="faKiwiBird" :mask="faMap" transform="shrink-12" />
        </template>

        妹法处理你的请求，好好想想自己的原因吧(ノ｀Д)ノ
      </ScreenEmptyState>
    </div>
  </section>
</template>

<script lang="ts" setup>
import { faKiwiBird, faMap } from '@fortawesome/free-solid-svg-icons'
import ScreenHeader from '@/components/ui/ScreenHeader.vue'
import ScreenEmptyState from '@/components/ui/ScreenEmptyState.vue'
//
</script>
