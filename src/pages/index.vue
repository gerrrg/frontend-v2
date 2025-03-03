<script setup lang="ts">
import { computed } from 'vue';
import { useRouter } from 'vue-router';

import HomePageHero from '@/components/heros/HomePageHero.vue';
import TokenSearchInput from '@/components/inputs/TokenSearchInput.vue';
import FeaturedProtocols from '@/components/sections/FeaturedProtocols.vue';
import PoolsTable from '@/components/tables/PoolsTable/PoolsTable.vue';
import usePoolFilters from '@/composables/pools/usePoolFilters';
import useStreamedPoolsQuery from '@/composables/queries/useStreamedPoolsQuery';
import useBreakpoints from '@/composables/useBreakpoints';
import useTokens from '@/composables/useTokens';
import useWeb3 from '@/services/web3/useWeb3';

// COMPOSABLES
const router = useRouter();
const { appNetworkConfig } = useWeb3();
const isElementSupported = appNetworkConfig.supportsElementPools;
const {
  selectedTokens,
  addSelectedToken,
  removeSelectedToken
} = usePoolFilters();

const {
  dataStates,
  result: investmentPools,
  loadMore,
  isLoadingMore
} = useStreamedPoolsQuery(selectedTokens);
const { upToMediumBreakpoint } = useBreakpoints();
const { priceQueryLoading } = useTokens();

const isInvestmentPoolsTableLoading = computed(
  () => dataStates.value['basic'] === 'loading' || priceQueryLoading.value
);

/**
 * METHODS
 */
function navigateToCreatePool() {
  router.push({ name: 'create-pool' });
}
</script>

<template>
  <HomePageHero />
  <div class="lg:container lg:mx-auto pt-10 md:pt-12">
    <BalStack vertical>
      <div class="px-4 lg:px-0">
        <h3 class="mb-3">{{ $t('investmentPools') }}</h3>
        <div
          class="flex flex-col md:flex-row w-full justify-between items-end lg:items-center"
        >
          <TokenSearchInput
            v-model="selectedTokens"
            @add="addSelectedToken"
            @remove="removeSelectedToken"
            class="w-full md:w-2/3"
          />
          <BalBtn
            @click="navigateToCreatePool"
            color="blue"
            size="sm"
            :class="{ 'mt-4': upToMediumBreakpoint }"
            :block="upToMediumBreakpoint"
          >
            {{ $t('createAPool.title') }}
          </BalBtn>
        </div>
      </div>
      <PoolsTable
        :data="investmentPools"
        :noPoolsLabel="$t('noPoolsFound')"
        :isLoadingMore="isLoadingMore"
        @loadMore="loadMore"
        :selectedTokens="selectedTokens"
        class="mb-8"
        :hiddenColumns="['migrate', 'actions', 'lockEndDate']"
        :columnStates="dataStates"
        :isPaginated="true"
        :isLoading="isInvestmentPoolsTableLoading"
      >
      </PoolsTable>
      <div v-if="isElementSupported" class="mt-16 p-4 lg:p-0">
        <FeaturedProtocols />
      </div>
    </BalStack>
  </div>
</template>
