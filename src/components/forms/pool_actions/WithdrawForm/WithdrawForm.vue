<script setup lang="ts">
import { computed, onBeforeMount, ref, toRef, watch } from 'vue';
import { useI18n } from 'vue-i18n';

// Components
import TokenInput from '@/components/inputs/TokenInput/TokenInput.vue';
import { isLessThanOrEqualTo, isRequired } from '@/lib/utils/validations';
import { Pool } from '@/services/pool/types';
import useWeb3 from '@/services/web3/useWeb3';

import ProportionalWithdrawalInput from './components/ProportionalWithdrawalInput.vue';
import WithdrawalTokenSelect from './components/WithdrawalTokenSelect.vue';
import WithdrawPreviewModal from './components/WithdrawPreviewModal/WithdrawPreviewModal.vue';
import WithdrawTotals from './components/WithdrawTotals.vue';
import useWithdrawalState from './composables/useWithdrawalState';
// Composables
import useWithdrawMath from './composables/useWithdrawMath';

/**
 * TYPES
 */
type Props = {
  pool: Pool;
};

/**
 * PROPS & EMITS
 */
const props = defineProps<Props>();

const showPreview = ref(false);

/**
 * COMPOSABLES
 */
const { t } = useI18n();

const {
  isProportional,
  tokenOut,
  tokenOutIndex,
  highPriceImpactAccepted,
  validInput,
  maxSlider,
  tokensOut,
  error,
  parseError,
  setError
} = useWithdrawalState(toRef(props, 'pool'));

const withdrawMath = useWithdrawMath(
  toRef(props, 'pool'),
  isProportional,
  tokenOut,
  tokenOutIndex
);

const {
  hasAmounts,
  highPriceImpact,
  singleAssetMaxes,
  tokenOutAmount,
  tokenOutPoolBalance,
  initMath,
  loadingAmountsOut
} = withdrawMath;

const {
  isWalletReady,
  startConnectWithInjectedProvider,
  isMismatchedNetwork
} = useWeb3();

/**
 * COMPUTED
 */
const hasAcceptedHighPriceImpact = computed((): boolean =>
  highPriceImpact.value ? highPriceImpactAccepted.value : true
);

const hasValidInputs = computed(
  (): boolean => validInput.value && hasAcceptedHighPriceImpact.value
);

const singleAssetRules = computed(() => [
  isLessThanOrEqualTo(tokenOutPoolBalance.value, t('exceedsPoolBalance'))
]);

/**
 * WATCHERS
 */
watch(isProportional, newVal => {
  // If user selects to withdraw all tokens proportionally
  // reset the slider to max.
  if (newVal) {
    initMath();
    maxSlider();
  }
});

/**
 * CALLBACKS
 */
onBeforeMount(() => {
  isProportional.value = true;
  initMath();
  maxSlider();
});
</script>

<template>
  <div>
    <ProportionalWithdrawalInput
      v-if="isProportional"
      :pool="pool"
      :tokenAddresses="tokensOut"
      :math="withdrawMath"
    />
    <TokenInput
      v-else
      :name="tokenOut"
      :address="tokenOut"
      v-model:amount="tokenOutAmount"
      v-model:isValid="validInput"
      :disableBalance="singleAssetMaxes[tokenOutIndex] === '-'"
      :customBalance="singleAssetMaxes[tokenOutIndex] || '0'"
      :rules="singleAssetRules"
      :balanceLabel="$t('singleTokenMax')"
      :balanceLoading="loadingAmountsOut"
      fixedToken
      disableNativeAssetBuffer
    >
      <template #tokenSelect>
        <WithdrawalTokenSelect :pool="pool" :initToken="tokenOut" />
      </template>
    </TokenInput>

    <WithdrawTotals :math="withdrawMath" class="mt-4" />

    <div
      v-if="highPriceImpact"
      class="border dark:border-gray-700 rounded-lg p-2 pb-2 mt-4"
    >
      <BalCheckbox
        v-model="highPriceImpactAccepted"
        :rules="[isRequired($t('priceImpactCheckbox'))]"
        name="highPriceImpactAccepted"
        size="sm"
        :label="$t('priceImpactAccept', [$t('withdrawing')])"
      />
    </div>

    <BalAlert
      v-if="error !== null"
      type="error"
      :title="parseError(error).title"
      :description="parseError(error).description"
      class="mt-4"
      block
      actionLabel="Dismiss"
      @actionClick="setError(null)"
    />

    <div class="mt-4">
      <BalBtn
        v-if="!isWalletReady"
        :label="$t('connectWallet')"
        color="gradient"
        block
        @click="startConnectWithInjectedProvider"
      />
      <BalBtn
        v-else
        :label="$t('preview')"
        color="gradient"
        :disabled="
          !hasAmounts ||
            !hasValidInputs ||
            isMismatchedNetwork ||
            loadingAmountsOut
        "
        block
        @click="showPreview = true"
      />
    </div>

    <teleport to="#modal">
      <WithdrawPreviewModal
        v-if="showPreview"
        :pool="pool"
        :math="withdrawMath"
        @close="showPreview = false"
      />
    </teleport>
  </div>
</template>
