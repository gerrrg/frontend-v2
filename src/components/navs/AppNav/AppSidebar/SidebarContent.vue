<script lang="ts" setup>
import { ref, watch } from 'vue';
import { useI18n } from 'vue-i18n';
import { useRouter } from 'vue-router';

import AppLogo from '@/components/images/AppLogo.vue';
import useApp from '@/composables/useApp';
import useConfig from '@/composables/useConfig';
import useDarkMode from '@/composables/useDarkMode';
import { sleep } from '@/lib/utils';
import useWeb3 from '@/services/web3/useWeb3';

/**
 * PROPS & EMITS
 */
const emit = defineEmits(['close']);

/**
 * COMPOSABLES
 */
const { darkMode, toggleDarkMode } = useDarkMode();
const { blockNumber } = useWeb3();
const { networkConfig } = useConfig();
const { version } = useApp();
const { t } = useI18n();
const router = useRouter();

/**
 * STATE
 */
const blockIcon = ref<HTMLDivElement>();

const navLinks = [
  { label: t('invest'), path: '/' },
  { label: t('trade'), path: '/trade' },
  { label: t('portfolio'), path: '/portfolio' },
  { label: 'veBAL', path: '/vebal' },
  { label: t('claim'), path: '/claim' }
];

const ecosystemLinks = [
  { label: t('build'), url: 'https://balancer.fi/build' },
  { label: t('blog'), url: 'https://medium.com/balancer-protocol' },
  { label: t('docs'), url: 'https://docs.balancer.fi/' },
  { label: t('governance'), url: 'https://vote.balancer.fi/#/' },
  { label: t('analytics'), url: 'https://dune.xyz/balancerlabs' },
  { label: t('forum'), url: 'https://forum.balancer.fi/' },
  {
    label: t('grants'),
    url: 'http://grants.balancer.community'
  }
];

const socialLinks = [
  { component: 'TwitterIcon', url: 'https://twitter.com/BalancerLabs' },
  { component: 'DiscordIcon', url: 'https://discord.balancer.fi/' },
  { component: 'MediumIcon', url: 'https://medium.com/balancer-protocol' },
  {
    component: 'YoutubeIcon',
    url: 'https://www.youtube.com/channel/UCBRHug6Hu3nmbxwVMt8x_Ow'
  },
  { component: 'GithubIcon', url: 'https://github.com/balancer-labs/' }
];

/**
 * METHODS
 */
async function navTo(path: string) {
  router.push(path);
  emit('close');
}

/**
 * WATCHERS
 */
watch(blockNumber, async () => {
  blockIcon.value?.classList.add('block-change');
  await sleep(300);
  blockIcon.value?.classList.remove('block-change');
});
</script>

<template>
  <div class="opacity-0 fade-in-delay">
    <div
      class="h-20 px-4 border-b border-gray-800 flex flex-col justify-center"
    >
      <AppLogo forceDark />
    </div>

    <div class="grid grid-col-1 text-lg mt-2">
      <div
        v-for="link in navLinks"
        :key="link.label"
        class="side-bar-link"
        @click="navTo(link.path)"
      >
        {{ link.label }}
      </div>
    </div>

    <div class="grid grid-col-1 text-sm mt-5">
      <span class="text-gray-500 px-4 pb-1 font-medium">Ecosystem</span>
      <BalLink
        v-for="link in ecosystemLinks"
        :key="link.url"
        :href="link.url"
        class="side-bar-link flex items-center"
        external
        noStyle
      >
        {{ link.label }}
        <BalIcon name="arrow-up-right" size="sm" class="ml-1 text-gray-500" />
      </BalLink>
    </div>

    <div class="mt-6 px-4">
      <div class="side-bar-btn mt-2" @click="toggleDarkMode">
        <MoonIcon v-if="!darkMode" class="mr-2" />
        <SunIcon v-else class="mr-2" />
        <span>{{ darkMode ? 'Light' : 'Dark' }} mode</span>
      </div>
    </div>

    <div class="mt-4 px-4 grid grid-rows-1 grid-flow-col auto-cols-min gap-2">
      <BalLink
        v-for="link in socialLinks"
        :key="link.component"
        :href="link.url"
        class="social-link"
        noStyle
        external
      >
        <component :is="link.component" />
      </BalLink>
      <BalLink
        href="mailto:contact@balancer.finance"
        class="social-link"
        noStyle
      >
        <EmailIcon />
      </BalLink>
    </div>

    <div class="mt-6 px-4 text-xs">
      <div class="flex items-center">
        <div
          ref="blockIcon"
          class="block-icon w-2 h-2 rounded-full bg-green-500"
        />
        <span class="ml-2 text-gray-300">
          {{ networkConfig.name }}: Block {{ blockNumber }}
        </span>
      </div>
      <BalLink
        :href="
          `https://github.com/balancer-labs/frontend-v2/releases/tag/${version}`
        "
        class="text-gray-300 flex items-center mt-2"
        external
        noStyle
      >
        App: v{{ version }}
        <BalIcon name="arrow-up-right" size="xs" class="ml-1" />
      </BalLink>
    </div>
  </div>
</template>

<style scoped>
.side-bar-link {
  @apply transition duration-300 p-4 py-1.5 hover:bg-gray-850 cursor-pointer;
}

.side-bar-btn {
  @apply flex items-center bg-gray-850 hover:bg-gray-800 rounded-lg p-2 cursor-pointer transition;
}

.social-link {
  @apply w-11 h-11 xs:w-12 xs:h-12  rounded-full bg-gray-850 hover:bg-gray-800 flex items-center justify-center text-white cursor-pointer;
}

.social-link > svg {
  @apply w-6 h-6;
  fill: white;
}

.block-icon {
  box-shadow: 0px 0px 3px 2px theme('colors.green.500');
  transition: box-shadow 0.3s ease-in-out;
}

.block-change {
  box-shadow: 0px 0px 6px 4px theme('colors.green.500');
}
</style>
