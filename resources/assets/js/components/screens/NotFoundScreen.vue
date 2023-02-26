<template>
  <section>
    <ScreenHeader layout="collapsed">什么也妹有哦</ScreenHeader>

    <div class="main-scroll-wrap">
      <ScreenEmptyState>
        <template #icon>
          <icon :icon="faKiwiBird" :mask="faMap" transform="shrink-12" />
        </template>

        妹有发现请求内容
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
