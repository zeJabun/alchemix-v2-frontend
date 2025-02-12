<script>
  import { _ } from 'svelte-i18n';
  import { slide } from 'svelte/transition';
  import ToggleSwitch from '@components/elements/ToggleSwitch.svelte';
  import InputNumber from '@components/elements/inputs/InputNumber.svelte';
  import Button from '@components/elements/Button.svelte';
  import settings from '@stores/settings';

  export let maxLoss;
  export let preset = '0.3';

  const typeOfLosses = Object.freeze({
    '0.3': 0,
    '0.5': 1,
    '1': 2,
  });

  const lossPresets = {
    [typeOfLosses['0.3']]: 0.3,
    [typeOfLosses['0.5']]: 0.5,
    [typeOfLosses['1']]: 1,
  };

  const maximumLossInputPref = {
    min: 0,
    max: 100,
    multiplier: 1000,
    safeMin: 0.3,
    saneMax: 5,
  };

  let currentPreset = typeOfLosses[preset];

  let useCustomAmount = false;
  let maximumLossInput = 0;

  maxLoss = lossPresets[typeOfLosses['0.3']] * maximumLossInputPref.multiplier;

  $: maxLoss = useCustomAmount
    ? maximumLossInput < 100
      ? maximumLossInput * maximumLossInputPref.multiplier
      : 100 * maximumLossInputPref.multiplier
    : lossPresets[currentPreset] * maximumLossInputPref.multiplier;
</script>

<div class="flex flex-col space-y-2 w-full">
  <div class="flex flex-row w-full">
    <p class="flex-auto {$settings.invertColors ? 'text-lightgrey10inverse' : 'text-lightgrey10'} text-sm">
      {$_('max_slippage')}:
    </p>
    <ToggleSwitch
      label="{$_('custom_amount')}"
      on:toggleChange="{() => (useCustomAmount = !useCustomAmount)}"
    />
  </div>

  {#if useCustomAmount}
    <div
      class="flex {$settings.invertColors ? 'bg-grey3inverse' : 'bg-grey3'} rounded border {maximumLossInput >
      maximumLossInputPref.max
        ? 'border-red3'
        : (maximumLossInput && maximumLossInput < maximumLossInputPref.safeMin) ||
          maximumLossInput >= maximumLossInputPref.saneMax
        ? 'text-orange1'
        : `${$settings.invertColors ? 'border-grey3inverse' : 'border-grey3'}`}"
      transition:slide|local
    >
      <div class="w-full">
        <InputNumber
          id="underlyingInput"
          bind:value="{maximumLossInput}"
          placeholder="0-100%"
          class="w-full rounded appearance-none text-xl text-right h-full p-4 {$settings.invertColors
            ? 'bg-grey3inverse'
            : 'bg-grey3'} {maximumLossInput > maximumLossInputPref.max
            ? 'text-red3'
            : maximumLossInput < maximumLossInputPref.safeMin ||
              maximumLossInput >= maximumLossInputPref.saneMax
            ? 'text-orange1'
            : 'text-lightgrey5'}"
        />
      </div>
      <div class="flex flex-col">
        <Button
          label="MAX"
          width="w-full"
          fontSize="text-xs"
          textColor="{$settings.invertColors ? 'lightgrey10inverse' : 'lightgrey10'}"
          backgroundColor="{$settings.invertColors ? 'grey3inverse' : 'grey3'}"
          borderSize="0"
          height="h-10"
          on:clicked="{() => {
            maximumLossInput = maximumLossInputPref.max;
          }}"
        />
        <Button
          label="CLEAR"
          width="w-max"
          fontSize="text-xs"
          textColor="{$settings.invertColors ? 'lightgrey10inverse' : 'lightgrey10'}"
          backgroundColor="{$settings.invertColors ? 'grey3inverse' : 'grey3'}"
          borderSize="0"
          height="h-10"
          on:clicked="{() => {
            maximumLossInput = '';
          }}"
        />
      </div>
    </div>
    {#if maximumLossInput && maximumLossInput < maximumLossInputPref.safeMin}
      <p class="text-orange1 text-sm text-center" transition:slide|local>
        {$_('modals.slippage_below_safe')}
      </p>
    {/if}
    {#if maximumLossInput > maximumLossInputPref.max}
      <p class="text-red3 text-sm text-center" transition:slide|local>
        {$_('modals.slippage_above_max')}
      </p>
    {:else if maximumLossInput >= maximumLossInputPref.saneMax}
      <p class="text-orange1 text-sm text-center" transition:slide|local>
        {$_('modals.slippage_above_safe')}
      </p>
    {/if}
  {:else}
    <div class="flex flex-row w-full text-lg" transition:slide|local>
      <button
        class="border {$settings.invertColors
          ? 'border-grey5inverse'
          : 'border-grey5'} rounded-l w-full {currentPreset === typeOfLosses['0.3']
          ? `${$settings.invertColors ? 'bg-grey3inverse' : 'bg-grey3'}`
          : `${$settings.invertColors ? 'hover:bg-grey10inverse' : 'hover:bg-grey10'}`}"
        on:click="{() => (currentPreset = typeOfLosses['0.3'])}"
      >
        <p
          class="text-center h-full py-3 {currentPreset === typeOfLosses['0.3']
            ? 'opacity-100'
            : 'opacity-50 hover:opacity-100'}"
        >
          0.3%
        </p>
      </button>
      <button
        class="border-t border-b {$settings.invertColors
          ? 'border-grey5inverse'
          : 'border-grey5'} w-full {currentPreset === typeOfLosses['0.5']
          ? `${$settings.invertColors ? 'bg-grey3inverse' : 'bg-grey3'}`
          : `${$settings.invertColors ? 'hover:bg-grey10inverse' : 'hover:bg-grey10'}`}"
        on:click="{() => (currentPreset = typeOfLosses['0.5'])}"
      >
        <p
          class="text-center h-full py-3 {currentPreset === typeOfLosses['0.5']
            ? 'opacity-100'
            : 'opacity-50 hover:opacity-100'}"
        >
          0.5%
        </p>
      </button>
      <button
        class="border {$settings.invertColors
          ? 'border-grey5inverse'
          : 'border-grey5'} rounded-r w-full {currentPreset === typeOfLosses['1']
          ? `${$settings.invertColors ? 'bg-grey3inverse' : 'bg-grey3'}`
          : `${$settings.invertColors ? 'hover:bg-grey10inverse' : 'hover:bg-grey10'}`}"
        on:click="{() => (currentPreset = typeOfLosses['1'])}"
      >
        <p
          class="text-center h-full py-3 {currentPreset === typeOfLosses['1']
            ? 'opacity-100'
            : 'opacity-50 hover:opacity-100'}"
        >
          1%
        </p>
      </button>
    </div>
  {/if}
</div>
