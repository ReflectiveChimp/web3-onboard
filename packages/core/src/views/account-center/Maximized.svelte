<script lang="ts">
  import { _ } from 'svelte-i18n'
  import { fly } from 'svelte/transition'
  import { quartOut } from 'svelte/easing'
  import { wallets$ } from '../../streams.js'
  import en from '../../i18n/en.json'
  import WalletRow from './WalletRow.svelte'
  import plusCircleIcon from '../../icons/plus-circle.js'
  import arrowForwardIcon from '../../icons/arrow-forward.js'
  import connect from '../../connect.js'
  import disconnect from '../../disconnect.js'
  import { state } from '../../store/index.js'
  import WalletAppBadge from '../shared/WalletAppBadge.svelte'
  import { getDefaultChainStyles, unrecognizedChainStyle } from '../../utils.js'
  import SuccessStatusIcon from '../shared/SuccessStatusIcon.svelte'
  import NetworkSelector from '../shared/NetworkSelector.svelte'
  import caretLightIcon from '../../icons/caret-light.js'
  import warningIcon from '../../icons/warning.js'
  import questionIcon from '../../icons/question.js'
  import { updateAccountCenter } from '../../store/actions.js'
  import blocknative from '../../icons/blocknative.js'
  import DisconnectAllConfirm from './DisconnectAllConfirm.svelte'
  import { configuration } from '../../configuration.js'

  function disconnectAllWallets() {
    $wallets$.forEach(({ label }) => disconnect({ label }))
  }

  const { chains: appChains } = state.get()
  const { appMetadata } = configuration
  let disconnectConfirmModal = false
  let hideWalletRowMenu: () => void

  $: [primaryWallet] = $wallets$
  $: [connectedChain] = primaryWallet ? primaryWallet.chains : []

  $: validAppChain = appChains.find(({ id, namespace }) =>
    connectedChain
      ? id === connectedChain.id && namespace === connectedChain.namespace
      : false
  )

  $: defaultChainStyles = getDefaultChainStyles(
    connectedChain && connectedChain.id
  )

  const { position } = state.get().accountCenter
  const { device } = configuration
</script>

<style>
  .outer-container {
    background: var(
      --account-center-maximized-upper-background,
      var(--onboard-gray-600, var(--gray-600))
    );
    border-radius: var(--onboard-border-radius-3, var(--border-radius-3));
    width: 100%;
    filter: drop-shadow(0px 4px 16px rgba(178, 178, 178, 0.2));
    padding: 0 1px 1px 1px;
    pointer-events: auto;
  }

  .wallets-section {
    width: 100%;
    border-radius: var(--onboard-border-radius-3, var(--border-radius-3));
  }

  .p5 {
    padding: var(--onboard-spacing-5, var(--spacing-5));
  }

  .wallets {
    width: 100%;
    margin-bottom: 0.5rem;
  }

  .actions {
    color: var(--onboard-primary-400, var(--primary-400));
    padding-left: 2px;
  }

  .action-container {
    padding: 4px 12px 4px 8px;
    border-radius: 8px;
    transition: background-color 150ms ease-in-out;
  }

  .action-container:hover {
    background-color: var(
      --account-center-maximized-action-background-hover,
      rgba(146, 155, 237, 0.2)
    );
  }

  .plus-icon {
    width: 20px;
  }

  .arrow-forward {
    width: 20px;
  }

  .mt {
    margin-top: 0.25rem;
  }

  .action-text {
    font-size: var(--onboard-font-size-6, var(--font-size-6));
    line-height: var(--onboard-font-line-height-3, var(--font-line-height-3));
    margin-left: 0.5rem;
  }

  .background-blue {
    background: var(--onboard-primary-100, var(--primary-100));
  }

  .background-gray {
    background: var(--onboard-gray-100, var(--gray-100));
  }

  .background-yellow {
    background: var(--onboard-warning-100, var(--warning-100));
  }

  .network-container {
    border-radius: var(--onboard-border-radius-3, var(--border-radius-3));
    color: var(
      --account-center-maximized-network-section,
      var(--onboard-gray-500, var(--gray-500))
    );
  }

  .p5-5 {
    padding: 12px;
  }

  .network-selector-container {
    margin-left: 1rem;
    width: 100%;
  }

  .network-selector-label {
    font-size: var(--onboard-font-size-7, var(--font-size-7));
    line-height: var(--onboard-font-line-height-3, var(--font-line-height-3));
  }

  .app-info-container {
    background: var(
      --account-center-maximized-app-info-section,
      var(--onboard-white, var(--white))
    );
    border-radius: 16px;
    padding: 12px;
  }

  .app-name {
    font-weight: 700;
    font-size: var(--onboard-font-size-5, var(--font-size-5));
    line-height: var(--onboard-font-line-height-3, var(--font-line-height-3));
    color: var(--onboard-gray-600, var(--gray-600));
    margin-bottom: var(--onboard-spacing-5, var(--spacing-5));
    margin-top: 0;
  }

  .app-description {
    font-size: var(--onboard-font-size-7, var(--font-size-7));
    line-height: var(--onboard-font-line-height-3, var(--font-line-height-3));
    color: var(--onboard-gray-500, var(--gray-500));
    margin: 0;
  }

  .app-info {
    font-size: var(--onboard-font-size-7, var(--font-size-7));
    line-height: var(--onboard-font-line-height-3, var(--font-line-height-3));
    color: var(--onboard-gray-500, var(--gray-500));
  }
  .app-info-heading {
    color: var(--onboard-gray-600, var(--gray-600));
    font-weight: 700;
    margin-top: var(--onboard-spacing-5, var(--spacing-5));
    margin-bottom: var(--onboard-spacing-7, var(--spacing-7));
  }

  a {
    font-weight: 700;
  }

  .mt7 {
    margin-top: var(--onboard-spacing-7, var(--spacing-7));
  }

  .ml4 {
    margin-left: var(--onboard-spacing-4, var(--spacing-4));
  }

  .app-button {
    margin-top: var(--onboard-spacing-5, var(--spacing-5));
    color: var(
      --account-center-app-btn-text-color,
      var(--onboard-white, var(--white))
    );
    background: var(
      --account-center-app-btn-background,
      var(--onboard-gray-500, var(--gray-500))
    );
    font-family: var(--account-center-app-btn-font-family, inherit);
  }

  .powered-by-container {
    margin-top: 12px;
  }

  .powered-by {
    color: var(--onboard-gray-400, var(--gray-400));
    font-size: var(--onboard-font-size-7, var(--font-size-7));
    line-height: var(--onboard-font-line-height-3, var(--font-line-height-3));
  }
</style>

{#if disconnectConfirmModal}
  <DisconnectAllConfirm
    onClose={() => (disconnectConfirmModal = false)}
    onConfirm={disconnectAllWallets}
  />
{/if}

<div
  in:fly={{
    delay: position.includes('top') ? 100 : 0,
    duration: 600,
    y: position.includes('top') ? 56 : -76,
    easing: quartOut,
    opacity: 0
  }}
  on:click|stopPropagation={hideWalletRowMenu}
  class="outer-container"
>
  <!-- wallets section -->
  <div class="wallets-section">
    <!-- connected accounts -->
    <div class="p5">
      <div class="wallets">
        {#each $wallets$ as wallet, i (wallet.label)}
          <WalletRow
            bind:hideMenu={hideWalletRowMenu}
            {wallet}
            primary={i === 0}
          />
        {/each}
      </div>
      <!-- actions -->
      <div class="actions flex flex-column items-start">
        <!-- Hide for Mobile  -->
        {#if device.type === 'desktop'}
          <!-- connect another wallet -->
          <div
            on:click={() => connect()}
            class="action-container flex items-center pointer"
          >
            <div class="plus-icon flex items-center justify-center">
              {@html plusCircleIcon}
            </div>
            <span class="action-text"
              >{$_('accountCenter.connectAnotherWallet', {
                default: en.accountCenter.connectAnotherWallet
              })}</span
            >
          </div>

          <!-- disconnect all wallets -->
          <div
            on:click={() => (disconnectConfirmModal = true)}
            class="action-container flex items-center mt pointer"
          >
            <div class="arrow-forward flex items-center justify-center">
              {@html arrowForwardIcon}
            </div>
            <span class="action-text"
              >{$_('accountCenter.disconnectAllWallets', {
                default: en.accountCenter.disconnectAllWallets
              })}</span
            >
          </div>
        {/if}
      </div>
    </div>

    <!-- network section -->
    <div
      class="network-container shadow-1"
      class:background-blue={(validAppChain && validAppChain.icon) ||
        defaultChainStyles}
      class:background-yellow={!validAppChain}
      class:background-gray={validAppChain && !defaultChainStyles}
    >
      <div class="flex items-center p5-5">
        <!-- network icon -->
        <div class="relative flex">
          <WalletAppBadge
            size={32}
            padding={4}
            background="custom"
            color={!validAppChain
              ? '#FFAF00'
              : !validAppChain.icon
              ? '#EFF1FC'
              : undefined}
            customBackgroundColor={validAppChain
              ? validAppChain.color ||
                (defaultChainStyles && defaultChainStyles.color) ||
                unrecognizedChainStyle.color
              : '#FFE7B3'}
            border="transparent"
            radius={8}
            icon={validAppChain
              ? validAppChain.icon ||
                (defaultChainStyles && defaultChainStyles.icon) ||
                unrecognizedChainStyle.icon
              : warningIcon}
          />

          {#if validAppChain}
            <div
              style="right: -5px; bottom: -5px;"
              class="drop-shadow absolute"
            >
              <SuccessStatusIcon size={14} />
            </div>
          {/if}
        </div>

        <!-- network selector -->
        <div class="network-selector-container">
          <div class="network-selector-label">
            {$_('accountCenter.currentNetwork', {
              default: en.accountCenter.currentNetwork
            })}
          </div>
          <div on:click class="flex items-center" style=" width: 100%;">
            <NetworkSelector
              chains={appChains}
              colorVar="--account-center-maximized-network-selector-color"
              bold={true}
              selectIcon={caretLightIcon}
              parentCSSId="maximized_ac"
            />
          </div>
        </div>
      </div>

      <!-- app info section -->
      <div class="app-info-container">
        <div class="flex items-start">
          <!-- app icon -->
          <div class="relative flex">
            <WalletAppBadge
              size={32}
              padding={4}
              background="transparent"
              border="black"
              radius={8}
              icon={(appMetadata && appMetadata.icon) || questionIcon}
            />

            <div
              style="right: -5px; bottom: -5px;"
              class="drop-shadow absolute"
            >
              <SuccessStatusIcon size={14} color="blue" />
            </div>
          </div>

          <div class="ml4">
            <h4 class="app-name">
              {(appMetadata && appMetadata.name) || 'App Name'}
            </h4>
            <p class="app-description">
              {(appMetadata && appMetadata.description) ||
                'This app has not added a description.'}
            </p>
          </div>
        </div>

        <!-- app info -->
        {#if appMetadata && (appMetadata.gettingStartedGuide || appMetadata.explore)}
          <div class="app-info">
            <h4 class="app-info-heading">
              {$_('accountCenter.appInfo', {
                default: en.accountCenter.appInfo
              })}
            </h4>

            {#if appMetadata.gettingStartedGuide}
              <div class="flex justify-between items-center mt7">
                <div>
                  {$_('accountCenter.learnMore', {
                    default: en.accountCenter.learnMore
                  })}
                </div>
                <a
                  href={appMetadata.gettingStartedGuide}
                  target="_blank"
                  rel="noreferrer noopener"
                >
                  {$_('accountCenter.gettingStartedGuide', {
                    default: en.accountCenter.gettingStartedGuide
                  })}
                </a>
              </div>
            {/if}

            {#if appMetadata.explore}
              <div class="flex justify-between items-center mt7">
                <div>
                  {$_('accountCenter.smartContracts', {
                    default: en.accountCenter.smartContracts
                  })}
                </div>
                <a
                  href={appMetadata.explore}
                  target="_blank"
                  rel="noreferrer noopener"
                >
                  {$_('accountCenter.explore', {
                    default: en.accountCenter.explore
                  })}
                </a>
              </div>
            {/if}
          </div>
        {/if}

        <button
          class="app-button button-neutral-solid"
          on:click={() => updateAccountCenter({ expanded: false })}
          >{$_('accountCenter.backToApp', {
            default: en.accountCenter.backToApp
          })}</button
        >
        <a
          href="https://blocknative.com"
          target="_blank"
          rel="noopener noreferrer"
          class="flex justify-center items-center powered-by-container"
        >
          <span class="powered-by"
            >{$_('accountCenter.poweredBy', {
              default: en.accountCenter.poweredBy
            })}</span
          >
          <div class="flex items-center" style="width: 83px; margin-left: 4px;">
            {@html blocknative}
          </div>
        </a>
      </div>
    </div>
  </div>
</div>
