<script setup lang="ts">
import { uniqBy } from 'lodash';
import { computed, ref } from 'vue';
import { useI18n } from 'vue-i18n';

import PoolsTable from '@/components/tables/PoolsTable/PoolsTable.vue';
import useUserPoolsQuery from '@/composables/queries/useUserPoolsQuery';
import useStaking from '@/composables/staking/useStaking';
import { isL2 } from '@/composables/useNetwork';
import { isMigratablePool } from '@/composables/usePool';
import { bnum } from '@/lib/utils';
import { Pool, PoolWithShares } from '@/services/pool/types';
import useWeb3 from '@/services/web3/useWeb3';

import StakePreviewModal from '../../stake/StakePreviewModal.vue';

/** STATE */
const showStakeModal = ref(false);
const stakePool = ref<Pool | undefined>();

/** COMPOSABLES */
const {
  userData: {
    userGaugeShares,
    userLiquidityGauges,
    stakedPools,
    isLoadingUserStakingData,
    poolBoosts
  },
  setPoolAddress
} = useStaking();
const { isWalletReady, isWalletConnecting } = useWeb3();
const { t } = useI18n();

/** COMPUTED */
// a map of poolId-stakedBPT for the connected user
const stakedBalanceMap = computed(() => {
  const map: Record<string, string> = {};
  if (!userGaugeShares.value) return map;
  for (const gaugeShare of userGaugeShares.value) {
    map[gaugeShare.gauge.poolId] = gaugeShare.balance;
  }
  return map;
});

const noPoolsLabel = computed(() => {
  return isWalletReady.value || isWalletConnecting.value
    ? t('noUnstakedInvestments')
    : t('connectYourWallet');
});

// first retrieve all the pools the user has liquidity for
const { data: userPools, isLoading: isLoadingUserPools } = useUserPoolsQuery();

const partiallyStakedPools = computed(() => {
  const stakedPoolIds = stakedPools.value?.map(pool => pool.id);
  // The pools which are both staked, but also have BPT available for staking
  // NOTE: we are iterating through user pools here so we can access the bpt var
  return (userPools.value?.pools || [])
    .filter(pool => {
      return stakedPoolIds?.includes(pool.id);
    })
    .map(pool => {
      // calculate the staked percentage by using the staked balance
      // pulled from the gauge subgraph
      const stakedBalance = stakedBalanceMap.value[pool.id];
      const unstakedBalance = pool.bpt;
      const stakedPct = bnum(stakedBalance).div(
        bnum(stakedBalance).plus(unstakedBalance)
      );
      return {
        ...pool,
        stakedPct: stakedPct.toString(),
        stakedShares: calculateFiatValueOfShares(pool, stakedBalance),
        boost: poolBoosts.value[pool.id]
      };
    });
});

// Pools where there is no staked BPT at all
const unstakedPools = computed(() => {
  const availableGaugePoolIds = (userLiquidityGauges.value || []).map(
    gauge => gauge.poolId
  );
  return (userPools.value?.pools || [])
    .filter(pool => {
      return availableGaugePoolIds?.includes(pool.id);
    })
    .map(pool => ({
      ...pool,
      stakedPct: '0',
      stakedShares: '0'
    }));
});

const poolsToRender = computed(() => {
  const stakablePools = [...partiallyStakedPools.value, ...unstakedPools.value];
  const stakableUserPoolIds = stakablePools.map(pool => pool.id);
  const nonMigratableUserPools = (userPools.value?.pools || [])
    .filter(pool => !isMigratablePool(pool))
    .filter(pool => !stakableUserPoolIds.includes(pool.id));
  // now mash them together
  return uniqBy([...nonMigratableUserPools, ...stakablePools], pool => pool.id);
});

const hiddenColumns = ['poolVolume', 'poolValue', 'migrate', 'lockEndDate'];

/** METHODS */
function handleStake(pool: Pool) {
  setPoolAddress(pool.address);
  showStakeModal.value = true;
  stakePool.value = pool;
}

function calculateFiatValueOfShares(
  pool: PoolWithShares | Pool,
  stakedBalance: string
) {
  return bnum(pool.totalLiquidity)
    .div(pool.totalShares)
    .times((stakedBalance || '0').toString())
    .toString();
}

function handleModalClose() {
  showStakeModal.value = false;
}
</script>

<template>
  <div>
    <BalStack vertical spacing="sm">
      <h5 class="px-4 lg:px-0" v-if="!isL2">
        {{ $t('staking.unstakedPools') }}
      </h5>
      <PoolsTable
        :key="poolsToRender"
        :isLoading="isLoadingUserStakingData || isLoadingUserPools"
        :data="poolsToRender"
        :noPoolsLabel="noPoolsLabel"
        :hiddenColumns="hiddenColumns"
        @triggerStake="handleStake"
        showPoolShares
      />
    </BalStack>
    <StakePreviewModal
      :pool="stakePool"
      :isVisible="showStakeModal"
      @close="handleModalClose"
      action="stake"
    />
  </div>
</template>
