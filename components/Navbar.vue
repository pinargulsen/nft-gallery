<template>
  <nav
    role="navigation"
    aria-label="main navigation"
    class="navbar is-fixed-top is-spaced"
    :class="{
      'is-active': isMobileNavbarOpen,
    }">
    <div class="container items-center" :class="{ 'is-fluid': !isTouch }">
      <!-- BRAND -->
      <div class="navbar-brand">
        <nuxt-link
          v-if="!isMobile || !openMobileSearchBar"
          to="/"
          class="navbar-item logo nuxt-link-active">
          <img
            :src="logoSrc"
            alt="First NFT market explorer on Kusama and Polkadot" />
        </nuxt-link>
        <div
          class="lg:!hidden flex flex-grow items-center justify-end"
          @click="closeBurgerMenu">
          <NeoButton
            v-show="isMobile && !openMobileSearchBar"
            class="square-40 mr-2"
            icon="magnifying-glass"
            @click="showMobileSearchBar" />

          <div v-show="!isMobile || openMobileSearchBar" class="w-full">
            <div
              class="is-fixed-top z-10 flex items-center justify-between p-2">
              <Search ref="mobilSearchRef" hide-filter class="flex-grow" />
              <NeoButton
                variant="text"
                class="p-3 is-shadowless border-0 capitalize sm:hidden"
                @click="hideMobileSearchBar">
                {{ $t('cancel') }}
              </NeoButton>
            </div>
          </div>
        </div>

        <!-- BURGER MENU -->
        <a
          role="button"
          class="navbar-burger sm:hidden"
          :class="{ 'is-active': isMobileNavbarOpen }"
          aria-label="menu"
          aria-expanded="false"
          data-target="MainNavbar"
          @click="showMobileNavbar">
          <span aria-hidden="true"></span>
          <span aria-hidden="true"></span>
          <span aria-hidden="true"></span>
        </a>
        <!-- END BURGER MENU -->
      </div>
      <!-- END BRAND -->

      <!-- MENU -->
      <div
        id="MainNavbar"
        class="navbar-menu"
        :class="{ 'is-active': isMobileNavbarOpen }">
        <!-- NAV START -->
        <div class="navbar-start">
          <div class="navbar-item is-expanded flex justify-center">
            <Search
              v-if="!isTouch"
              class="search-navbar flex-grow pb-0 max-lg:!hidden"
              hide-filter
              search-column-class="flex-grow" />
          </div>
        </div>
        <!-- END NAV START -->

        <!-- NAV END -->
        <div class="navbar-end">
          <nuxt-link to="/ahp/drops" rel="nofollow">
            <div class="navbar-item" data-testid="drops">
              {{ $t('drops.drops') }}

              <NeoIcon
                class="ml-1"
                icon="fire-flame-curved"
                pack="fass"
                variant="primary" />
            </div>
          </nuxt-link>

          <MobileExpandableSection
            v-slot="{ onCloseMobileSubMenu }"
            class="lg:!hidden"
            :title="$t('explore')">
            <NavbarExploreOptions
              @select="
                () => {
                  showMobileNavbar()
                  onCloseMobileSubMenu()
                }
              " />
          </MobileExpandableSection>
          <NavbarExploreDropdown
            class="navbar-explore custom-navbar-item max-lg:!hidden"
            data-testid="explore" />

          <a
            href="https://hello.kodadot.xyz"
            rel="nofollow noopener noreferrer"
            target="_blank"
            class="navbar-item"
            data-testid="learn">
            {{ $t('learn') }}
          </a>
          <CreateDropdown
            v-show="isCreateVisible"
            class="navbar-create custom-navbar-item ml-0"
            data-testid="create"
            :chain="urlPrefix"
            @select="showMobileNavbar" />

          <!-- commenting as part of #5889-->
          <!-- <StatsDropdown
          class="navbar-stats custom-navbar-item"
          data-testid="stats"
          :is-mobile="isMobile"
          :chain="urlPrefix" /> -->

          <MobileExpandableSection
            v-slot="{ onCloseMobileSubMenu }"
            class="lg:!hidden"
            no-padding
            :title="$t('chainSelect', [chainName])">
            <NavbarChainOptions
              @select="
                () => {
                  handleMobileChainSelect()
                  onCloseMobileSubMenu()
                }
              " />
          </MobileExpandableSection>
          <ChainSelectDropdown
            id="NavChainSelect"
            class="navbar-chain custom-navbar-item max-lg:!hidden"
            data-testid="chain-select" />

          <NotificationBoxButton
            v-if="account"
            data-testid="navbar-button-notification"
            :show-label="isTouch"
            @closeBurgerMenu="showMobileNavbar" />

          <ShoppingCartButton
            data-testid="navbar-button-cart"
            :show-label="isTouch"
            @closeBurgerMenu="showMobileNavbar" />

          <div class="lg:!hidden">
            <template v-if="!account">
              <MobileExpandableSection
                v-slot="{ onCloseMobileSubMenu }"
                class="mobile-language"
                :no-padding="true"
                :title="$t('profileMenu.language')"
                icon="globe">
                <MobileLanguageOption
                  @select="
                    () => {
                      showMobileNavbar()
                      onCloseMobileSubMenu()
                    }
                  " />
              </MobileExpandableSection>
              <ColorModeButton class="navbar-item" />
            </template>
            <div
              v-else
              class="navbar-item"
              @click.stop="openWalletConnectModal">
              <span>
                {{ $t('profile.page') }}
                <NeoIcon icon="user-circle" class="w-4 h-4" size="medium" />
              </span>
              <NeoIcon class="icon--right" icon="chevron-right" />
            </div>

            <div v-if="!account" id="NavProfile">
              <ConnectWalletButton
                class="button-connect-wallet"
                data-testid="navbar-button-connect-wallet"
                variant="connect"
                @closeBurgerMenu="showMobileNavbar" />
            </div>
          </div>

          <NavbarProfileDropdown
            id="NavProfile"
            class="max-lg:!hidden"
            :chain="urlPrefix"
            data-testid="navbar-profile-dropdown"
            @closeBurgerMenu="closeBurgerMenu" />
        </div>
        <!-- END NAV END -->
      </div>
      <!-- END MENU -->
    </div>
  </nav>
</template>

<script lang="ts" setup>
import { NeoButton, NeoIcon } from '@kodadot1/brick'
import { nextTick } from 'vue'
import { ConnectWalletModalConfig } from '@/components/common/ConnectWallet/useConnectWallet'
import ChainSelectDropdown from '@/components/navbar/ChainSelectDropdown.vue'
import CreateDropdown from '@/components/navbar/CreateDropdown.vue'
import MobileExpandableSection from '@/components/navbar/MobileExpandableSection.vue'
import MobileLanguageOption from '@/components/navbar/MobileLanguageOption.vue'
import NavbarChainOptions from '@/components/navbar/NavbarChainOptions.vue'
import NavbarExploreOptions from '@/components/navbar/NavbarExploreOptions.vue'
import NotificationBoxButton from '@/components/navbar/NotificationBoxButton.vue'
import Search from '@/components/search/Search.vue'
import ConnectWalletButton from '@/components/shared/ConnectWalletButton.vue'

import { useIdentityStore } from '@/stores/identity'
import { getChainNameByPrefix } from '@/utils/chain'
import { createVisible } from '@/utils/config/permission.config'
import ShoppingCartButton from './navbar/ShoppingCartButton.vue'

const { neoModal } = useProgrammatic()
const openMobileSearchBar = ref(false)
const lastScrollPosition = ref(0)
const isBurgerMenuOpened = ref(false)
const { isMobile, isMobileOrTablet: isTouch } = useDevice()
const { urlPrefix } = usePrefix()
const { isDarkMode } = useTheme()
const identityStore = useIdentityStore()
const isMobileNavbarOpen = ref(false)
const updateAuthBalanceTimer = ref()

const mobilSearchRef = ref<{ focusInput: () => void } | null>(null)

const account = computed(() => identityStore.getAuthAddress)

const isCreateVisible = computed(() => createVisible(urlPrefix.value))

const logoSrc = computed(() => {
  const variant = isTouch ? 'Koda' : 'Koda_Beta'
  const color = isDarkMode.value ? '_dark' : ''
  return `/${variant}${color}.svg`
})

const handleMobileChainSelect = () => {
  showMobileNavbar()
}

const closeAllModals = () => neoModal.closeAll()

const openWalletConnectModal = (): void => {
  showMobileNavbar()

  closeAllModals()
  neoModal.open({
    ...ConnectWalletModalConfig,
    ...(isMobile ? { animation: 'none' } : {}),
  })
}

const showMobileNavbar = () => {
  closeAllModals()

  document.body.classList.toggle('is-clipped')
  isMobileNavbarOpen.value = !isMobileNavbarOpen.value
  if (!isMobileNavbarOpen.value) {
    document.documentElement.scrollTop = lastScrollPosition.value
  }
}

const closeBurgerMenu = () => {
  isBurgerMenuOpened.value = false
}

watch([isBurgerMenuOpened], () => {
  setBodyScroll(!isBurgerMenuOpened.value)
})

const setBodyScroll = (allowScroll: boolean) => {
  nextTick(() => {
    const body = document.querySelector('body') as HTMLBodyElement
    if (allowScroll) {
      body.classList.remove('is-clipped')
    } else {
      body.classList.add('is-clipped')
    }
  })
}

const showMobileSearchBar = () => {
  openMobileSearchBar.value = true
  nextTick(() => {
    mobilSearchRef.value?.focusInput()
  })
  setBodyScroll(false)
}

const hideMobileSearchBar = () => {
  openMobileSearchBar.value = false
  setBodyScroll(true)
}

const chainName = computed(() => getChainNameByPrefix(urlPrefix.value))

const updateAuthBalance = () => {
  account.value && identityStore.fetchBalance({ address: account.value })
}

onMounted(() => {
  document.body.style.overflowY = 'initial'
  document.body.className = 'has-navbar-fixed-top has-spaced-navbar-fixed-top'
  updateAuthBalanceTimer.value = setInterval(updateAuthBalance, 30000)
})

onBeforeUnmount(() => {
  setBodyScroll(true)
  document.documentElement.classList.remove('is-clipped-touch')
  clearInterval(updateAuthBalanceTimer.value)
})
</script>

<style lang="scss" scoped>
.square-40 {
  width: 40px;
  height: 40px;
}

:deep(.navbar-explore) {
  .navbar-item {
    height: 4.5rem;
  }
  .o-drop__menu {
    margin: 0;
  }
  .o-drop__item {
    padding: 1.5rem 2rem;
    min-width: 18.75rem;

    &:hover {
      background-color: unset;
    }
  }
}
</style>
