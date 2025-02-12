<script>
  import { onDestroy, onMount } from 'svelte';
  import { fade } from 'svelte/transition';
  import { globalHistory } from 'svelte-routing/src/history';
  import { _ } from 'svelte-i18n';
  import Wallet from './Wallet.svelte';
  import { routerGuard } from '@helpers/routerGuard';
  import { sentinelStore, networkStore } from '@stores/v2/alcxStore';
  import settings from '@stores/settings';
  import { sidebarSetup } from '@stores/sidebarSetup';
  import secret from '@stores/secret';

  let pathname = window.location.pathname.slice(1);
  let unsub;

  const updatePath = (_pathname) => {
    pathname = _pathname;
    routerGuard(_pathname);
  };

  onMount(() => {
    unsub = globalHistory.listen(({ location }) => {
      pathname = location.pathname.slice(1);
    });
  });

  onDestroy(() => {
    unsub();
  });
</script>

<div class="relative flex items-center justify-between">
  <p class="flex flex-1 uppercase tracking-wider font-medium text-xs opacity-30 mb-4">
    {$_('wallet')}
  </p>
</div>
<Wallet />

<p class="uppercase tracking-wider font-medium text-xs opacity-30 my-4">
  {$_('navigation')}
</p>
<ul>
  {#each sidebarSetup()
    .filter((key) => key.label !== 'sentinel')
    .filter((key) => key.label !== 'Cows')
    .filter((key) => key.supportedChains.includes($networkStore)) as sidebarItem}
    <li
      class="p-4 rounded-xl mb-5 cursor-pointer flex justify-between transition-opacity {pathname ===
      `${sidebarItem.path}`
        ? `${$settings.invertColors ? 'bg-grey10inverse' : 'bg-grey10'} opacity-100`
        : 'opacity-40'} hover:{$settings.invertColors ? 'bg-grey10inverse' : 'bg-grey10'} hover:opacity-100"
      on:click="{() => updatePath(sidebarItem.path)}"
    >
      <span>{$_(sidebarItem.label)}</span>
      <img
        src="images/icons/{sidebarItem.icon}"
        class="w-7 h-7 {$settings.invertColors ? 'invertIcons' : ''}"
        alt="{sidebarItem.label}"
      />
    </li>
  {/each}
  {#each sidebarSetup()
    .filter((key) => key.label === 'sentinel')
    .filter((key) => key.supportedChains.includes($networkStore)) as sidebarItem}
    {#if $sentinelStore}
      <li
        class="p-4 rounded-xl mb-5 cursor-pointer flex justify-between transition-opacity {pathname ===
        `${sidebarItem.path}`
          ? `${$settings.invertColors ? 'bg-grey10inverse' : 'bg-grey10'} opacity-100`
          : 'opacity-40'} hover:{$settings.invertColors ? 'bg-grey10inverse' : 'bg-grey10'} hover:opacity-100"
        on:click="{() => updatePath(sidebarItem.path)}"
        transition:fade|local
      >
        <span>{$_(sidebarItem.label)}</span>
        <img
          src="images/icons/{sidebarItem.icon}"
          class="w-7 h-7 {$settings.invertColors ? 'invertIcons' : ''}"
          alt="{sidebarItem.label}"
        />
      </li>
    {/if}
  {/each}
  {#each sidebarSetup().filter((key) => key.label === 'Cows') as sidebarItem}
    {#if $secret.unlocked}
      <li
        class="p-4 rounded-xl mb-5 cursor-pointer flex justify-between transition-opacity {pathname.slice(
          1,
        ) === `${sidebarItem.path}`
          ? `${$settings.invertColors ? 'bg-grey10inverse' : 'bg-grey10'} opacity-100`
          : 'opacity-40'} hover:{$settings.invertColors ? 'bg-grey10inverse' : 'bg-grey10'} hover:opacity-100"
        on:click="{() => routerGuard(sidebarItem.path)}"
        transition:fade|local
      >
        <span>{$_(sidebarItem.label)}</span>
        <img
          src="images/icons/{sidebarItem.icon}"
          class="w-7 h-7 {$settings.invertColors ? 'invertIcons' : ''}"
          alt="{sidebarItem.label}"
        />
      </li>
    {/if}
  {/each}
</ul>
