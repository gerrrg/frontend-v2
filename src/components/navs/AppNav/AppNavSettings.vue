<template>
  <div>
    <div class="p-4 border-b dark:border-gray-900">
      <div class="flex justify-between items-center mb-6">
        <h5 v-text="$t('account')" class="leading-none tracking-tight" />
        <div class="flex items-center gap-2">
          <BalBtn color="gray" size="xs" @click="toggleWalletSelectModal">
            {{ $t('change') }}
          </BalBtn>
          <div v-if="!hideDisconnect">
            <BalBtn
              outline
              color="gray"
              size="xs"
              @click="disconnectWallet"
              class="capitalize"
            >
              {{ $t('disconnect') }}
            </BalBtn>
          </div>
        </div>
      </div>
      <div class="flex mt-1 mb-1">
        <div class="flex">
          <div class="relative">
            <Avatar :iconURI="profile?.avatar" :address="account" :size="44" />
            <div class="connector-icon-wrapper">
              <img
                :src="connectorLogo"
                class="p-0.5 w-5 h-5 absolute bottom-0 right-0 flex items-center justify-center bg-white rounded-full"
              />
            </div>
          </div>
          <div class="ml-2">
            <div class="address flex items-baseline">
              <div
                class="font-bold text-black dark:text-white"
                v-text="_shorten(account)"
              />
              <div class="ml-3 flex">
                <BalTooltip width="auto">
                  <template v-slot:activator>
                    <BalBtn
                      circle
                      color="gray"
                      size="xs"
                      flat
                      @click="copyAddress"
                    >
                      <BalIcon v-if="copiedAddress" name="check" size="xs" />
                      <BalIcon v-else name="clipboard" size="xs" />
                    </BalBtn>
                  </template>
                  <div
                    v-text="copiedAddress ? $t('copied') : $t('copyAddress')"
                    class="text-center"
                  />
                </BalTooltip>
                <BalBtn
                  circle
                  flat
                  color="gray"
                  size="xs"
                  tag="a"
                  :href="explorer.addressLink(account)"
                  target="_blank"
                  rel="noreferrer"
                  class="ml-2"
                >
                  <BalIcon name="arrow-up-right" size="xs" />
                </BalBtn>
              </div>
            </div>
            <div class="text-sm">{{ connectorName }}</div>
          </div>
        </div>
      </div>
    </div>
    <div class="hidden px-4 mt-4">
      <span v-text="$t('theme')" class="font-medium mb-2" />
      <div class="flex mt-1">
        <div
          class="option w-16 mr-2 py-1.5 flex items-center justify-center border rounded-xl cursor-pointer"
          :class="{ active: !appDarkMode }"
          @click="setDarkMode(false)"
        >
          <BalIcon name="sun" size="sm" />
        </div>
        <div
          class="option w-16 mr-2 py-1.5 flex items-center justify-center border rounded-xl cursor-pointer"
          :class="{ active: appDarkMode }"
          @click="setDarkMode(true)"
        >
          <BalIcon name="moon" size="sm" />
        </div>
      </div>
    </div>
    <div class="px-4 mt-4">
      <div class="flex items-baseline">
        <span v-text="$t('slippageTolerance')" class="font-medium mb-2" />
        <BalTooltip>
          <template v-slot:activator>
            <BalIcon name="info" size="xs" class="ml-1 text-gray-400 -mb-px" />
          </template>
          <div v-html="$t('marketConditionsWarning')" />
        </BalTooltip>
      </div>
      <AppSlippageForm class="mt-1" />
    </div>
    <div v-if="isEIP1559SupportedNetwork" class="px-4 mt-6">
      <div class="flex items-baseline">
        <span v-text="$t('transactionType')" class="font-medium mb-2" />
        <BalTooltip>
          <template v-slot:activator>
            <BalIcon name="info" size="xs" class="ml-1 text-gray-400 -mb-px" />
          </template>
          <div v-text="$t('ethereumTxTypeTooltip')" />
        </BalTooltip>
      </div>
      <BalBtnGroup
        :options="ethereumTxTypeOptions"
        v-model="ethereumTxType"
        @update:modelValue="setEthereumTxType"
      />
    </div>
    <div
      v-if="ENABLE_LEGACY_TRADE_INTERFACE && isGnosisSupportedNetwork"
      class="px-4 mt-6"
    >
      <div class="flex items-baseline">
        <span v-text="$t('tradeInterface')" class="font-medium mb-2" />
        <BalTooltip>
          <template v-slot:activator>
            <BalIcon name="info" size="xs" class="ml-1 text-gray-400 -mb-px" />
          </template>
          <div v-text="$t('tradeInterfaceTooltip')" class="w-52" />
        </BalTooltip>
      </div>
      <BalBtnGroup
        :options="tradeInterfaceOptions"
        v-model="appTradeInterface"
        @update:modelValue="setTradeInterface"
      />
      <div class="flex mt-1"></div>
    </div>
    <div
      class="network p-4 mt-4 text-sm border-t dark:border-gray-900 rounded-b-xl"
    >
      <div v-text="$t('network')" />
      <div class="flex items-baseline">
        <div :class="['w-2 h-2 mr-1 rounded-full', networkColorClass]"></div>
        {{ isUnsupportedNetwork ? $t('unsupportedNetwork') : networkName }}
      </div>
    </div>
  </div>
</template>

<script>
import { Network } from '@balancer-labs/sdk';
import { computed, defineComponent, reactive, toRefs } from 'vue';
import { useStore } from 'vuex';

import AppSlippageForm from '@/components/forms/AppSlippageForm.vue';
import Avatar from '@/components/images/Avatar.vue';
import { ENABLE_LEGACY_TRADE_INTERFACE } from '@/composables/trade/constants';
import useEthereumTxType from '@/composables/useEthereumTxType';
import {
  ethereumTxTypeOptions,
  tradeInterfaceOptions
} from '@/constants/options';
import { GP_SUPPORTED_NETWORKS } from '@/services/gnosis/constants';
import useWeb3 from '@/services/web3/useWeb3';
import {
  getConnectorLogo,
  getConnectorName
} from '@/services/web3/web3.plugin';
import { TradeInterface } from '@/store/modules/app';

export default defineComponent({
  components: {
    AppSlippageForm,
    Avatar
  },

  setup() {
    // COMPOSABLES
    const store = useStore();
    const {
      explorerLinks,
      account,
      profile,
      disconnectWallet,
      toggleWalletSelectModal,
      connector,
      provider,
      isEIP1559SupportedNetwork,
      userNetworkConfig,
      appNetworkConfig,
      isUnsupportedNetwork
    } = useWeb3();
    const { ethereumTxType, setEthereumTxType } = useEthereumTxType();

    // DATA
    const data = reactive({
      tradeInterfaceOptions,
      copiedAddress: false
    });

    // COMPUTED
    const networkColorClass = computed(() => {
      let color = 'green';

      if (isUnsupportedNetwork.value) {
        color = 'red';
      } else {
        switch (userNetworkConfig.value?.chainId) {
          case Network.KOVAN:
            color = 'purple';
            break;
          case Network.ROPSTEN:
            color = 'pink';
            break;
          case Network.RINKEBY:
            color = 'yellow';
            break;
          case Network.GÖRLI:
            color = 'blue';
            break;
        }
      }

      return `bg-${color}-500 dark:bg-${color}-400`;
    });
    const networkName = computed(() => userNetworkConfig.value?.name);
    const appLocale = computed(() => store.state.app.locale);
    const appDarkMode = computed(() => store.state.app.darkMode);
    const appTradeInterface = computed(() => store.state.app.tradeInterface);
    const connectorName = computed(() =>
      getConnectorName(connector.value?.id, provider.value)
    );
    const connectorLogo = computed(() =>
      getConnectorLogo(connector.value?.id, provider.value)
    );
    const hideDisconnect = computed(() => connector.value?.id == 'gnosis');
    const isGnosisSupportedNetwork = computed(() =>
      GP_SUPPORTED_NETWORKS.includes(appNetworkConfig.chainId)
    );

    // METHODS
    const setDarkMode = val => store.commit('app/setDarkMode', val);
    const setLocale = locale => store.commit('app/setLocale', locale);

    const setTradeInterface = tradeInterface =>
      store.commit('app/setTradeInterface', tradeInterface);

    function copyAddress() {
      navigator.clipboard.writeText(account.value);
      data.copiedAddress = true;

      setTimeout(() => {
        data.copiedAddress = false;
      }, 2 * 1000);
    }

    return {
      // data
      ...toRefs(data),
      TradeInterface,
      ENABLE_LEGACY_TRADE_INTERFACE,
      // computed
      account,
      profile,
      appTradeInterface,
      networkName,
      networkColorClass,
      appLocale,
      appDarkMode,
      connectorName,
      connectorLogo,
      hideDisconnect,
      isEIP1559SupportedNetwork,
      isGnosisSupportedNetwork,
      isUnsupportedNetwork,
      // methods
      disconnectWallet,
      toggleWalletSelectModal,
      setDarkMode,
      setLocale,
      setTradeInterface,
      copyAddress,
      explorer: explorerLinks,
      ethereumTxType,
      setEthereumTxType,
      ethereumTxTypeOptions
    };
  }
});
</script>

<style scoped>
.address {
  @apply text-blue-500;
  font-variant-ligatures: no-contextual;
}

.option:hover {
  @apply text-blue-500 border-blue-500;
}

.option.active {
  @apply text-blue-500 border-blue-500;
}

.slippage-input {
  @apply bg-white;
}

.slippage-input.active {
  @apply text-blue-500 border-blue-500;
}
</style>
