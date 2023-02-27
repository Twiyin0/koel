<template>
  <section class="text-secondary">
    <h1>Last.fm扩展</h1>

    <div v-if="useLastfm" data-testid="lastfm-integrated">
      <p>
        外卖夜安装了 Last.fm. 扩展
        <span v-if="connected">
          看来你也已连接 Last.fm 帐户了呢 - 完美！
        </span>
        <span v-else>不过，你似乎还没连接到 Last.fm 帐户呢</span>
      </p>
      <p>
        连接Koel和您的 Last.fm 帐户可以下面这个
        <a
          class="text-highlight"
          href="https://www.last.fm/about/trackmymusic"
          rel="noopener"
          target="_blank"
        >scrobbling</a>的功能（我也不知道是什么）.
      </p>
      <div class="buttons">
        <Btn class="connect" @click.prevent="connect">
          <icon :icon="faLastfm" />
          {{ connected ? 'Reconnect' : 'Connect' }}
        </Btn>

        <Btn v-if="connected" class="disconnect" gray @click.prevent="disconnect">断开连接</Btn>
      </div>
    </div>

    <div v-else data-testid="lastfm-not-integrated">
      <p>
        我们好像妹有安装 Last.fm 扩展欸.
        <span v-if="isAdmin" data-testid="lastfm-admin-instruction">
          可以看
          <a href="https://docs.koel.dev/3rd-party.html#last-fm" class="text-highlight" target="_blank">Koel’s Wiki</a>
          来安装此扩展
        </span>
        <span v-else data-testid="lastfm-user-instruction">
        然后叫管理启用该扩展
        </span>
      </p>
    </div>
  </section>
</template>

<script lang="ts" setup>
import { faLastfm } from '@fortawesome/free-brands-svg-icons'
import { computed, defineAsyncComponent } from 'vue'
import { authService, http } from '@/services'
import { forceReloadWindow } from '@/utils'
import { useAuthorization, useThirdPartyServices } from '@/composables'

const Btn = defineAsyncComponent(() => import('@/components/ui/Btn.vue'))

const { currentUser, isAdmin } = useAuthorization()
const { useLastfm } = useThirdPartyServices()

const connected = computed(() => Boolean(currentUser.value.preferences!.lastfm_session_key))

/**
 * Connect the current user to Last.fm.
 * This method opens a new window.
 * Koel will reload once the connection is successful.
 */
const connect = () => window.open(
  `${window.BASE_URL}lastfm/connect?api_token=${authService.getApiToken()}`,
  '_blank',
  'toolbar=no,titlebar=no,location=no,width=1024,height=640'
)

const disconnect = async () => {
  await http.delete('lastfm/disconnect')
  forceReloadWindow()
}
</script>

<style lang="scss" scoped>
.buttons {
  margin-top: 1.25rem;

  > * + * {
    margin-left: 0.5rem;
  }

  .connect {
    background: #d31f27; // Last.fm color yo!
  }
}
</style>
