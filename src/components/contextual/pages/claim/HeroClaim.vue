<script setup lang="ts"></script>

<template>
  <div class="bg">
    <div class="w-full hero-container">
      <div class="hero-content">
        <div class="hero-text fade-in-slow">
          <h1 class="title mb-3 text-white text-shadow">
            {{ $t('claimHero.title') }}
          </h1>
          <p class="body-text text-shadow">
            {{ $t('claimHero.description') }}
          </p>
        </div>
      </div>
      <div class="hero-graphic-container">
        <div
          class="grid grid-cols-2 sm:grid-cols-4 gap-8 sm:gap-4 xl:gap-12 items-center w-full fade-in-slow"
        >
          <div class="tip group">
            <img
              width="144"
              height="144"
              class="tip-icon"
              src="@/assets/images/icons/claims-header/add-liquidity.svg"
            />
            <p class="tip-label font-semibold text-shadow">
              {{ $t('claimHero.tipLabel.addLiquidity') }}
              <BalTooltip
                iconSize="xs"
                textAlign="left"
                class="relative top-px"
                iconClass="text-white"
                width="60"
              >
                {{ $t('claimHero.tips.addLiquidity') }}
              </BalTooltip>
            </p>
          </div>
          <div class="tip group">
            <img
              width="144"
              height="144"
              class="tip-icon"
              src="@/assets/images/icons/claims-header/stake.svg"
            />
            <p class="tip-label font-semibold text-shadow">
              {{ $t('claimHero.tipLabel.stake') }}
              <BalTooltip
                iconSize="xs"
                textAlign="left"
                class="relative top-px"
                iconClass="text-white"
                width="60"
              >
                {{ $t('claimHero.tips.stake') }}
              </BalTooltip>
            </p>
          </div>
          <div class="tip group">
            <img
              width="144"
              height="144"
              class="tip-icon"
              src="@/assets/images/icons/claims-header/vebal.svg"
            />
            <p class="tip-label font-semibold text-shadow">
              {{ $t('claimHero.tipLabel.boost') }}
              <BalTooltip
                iconSize="xs"
                textAlign="left"
                class="relative top-px"
                iconClass="text-white"
                width="60"
              >
                {{ $t('claimHero.tips.boost') }}
              </BalTooltip>
            </p>
          </div>
          <div class="tip group">
            <img
              width="144"
              height="144"
              class="tip-icon"
              src="@/assets/images/icons/claims-header/claim.svg"
            />
            <p class="tip-label font-semibold text-shadow">
              {{ $t('claimHero.tipLabel.claim') }}
              <BalTooltip
                iconSize="xs"
                textAlign="left"
                class="relative top-px"
                iconClass="text-white"
                width="60"
              >
                {{ $t('claimHero.tips.claim') }}
              </BalTooltip>
            </p>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.bg {
  @apply bg-gradient-to-r from-pink-500 to-yellow-500 relative;
  background-image: url('/images/backgrounds/claims-header.svg');
  @apply bg-no-repeat bg-cover bg-center;
}

.hero-container {
  @apply grid grid-cols-1 lg:grid-cols-3 lg:container lg:mx-auto;
}

.hero-graphic-container {
  @apply w-full px-4 py-8 xl:p-8 flex items-center lg:col-span-2;
}

.hero-content {
  @apply px-4 pt-12 lg:p-8 lg:pl-0 flex items-center w-full relative;
}

.hero-text {
  @apply opacity-0;
}

.body-text {
  @apply text-white pb-0 font-medium;
  text-shadow: 0px 1px 0px rgb(0 0 0 / 15%), 4px 4px 8px rgb(0 0 0 / 15%);
}
.tip {
  @apply flex flex-col items-center text-center group;
}
.tip-icon {
  @apply pb-4 transition-transform duration-300;
}
.tip-label {
  @apply text-white leading-5 transition-colors;
}

.tip:hover .tip-icon {
  /* Tailwind class -translate-y-2 not working for some reason here */
  transform: translateY(-8px);
}

.tip-number {
  @apply block pb-1;
}
</style>
