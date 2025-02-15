<script lang="ts">
import { faCircleExclamation, faXmark } from '@fortawesome/free-solid-svg-icons';
import { createEventDispatcher } from 'svelte';
import Fa from 'svelte-fa';

export let placeholder: string | undefined = undefined;
export let id: string | undefined = undefined;
export let name: string | undefined = undefined;
export let value: string | undefined = undefined;
export let readonly: boolean = false;
export let required: boolean = false;
export let clearable: boolean = false;
export let disabled: boolean = false;
export let error: string | undefined = undefined;

export let element: HTMLInputElement | undefined = undefined;

const dispatch = createEventDispatcher();

// clear the value if the parent doesn't override
async function onClear() {
  if (dispatch('action', { cancelable: true })) {
    value = '';
    if (element) {
      // need to trigger normal input event for on:input listeners
      element.value = value;
      element.dispatchEvent(new InputEvent('input'));
    }
  }
}
</script>

<div class="flex flex-col w-full">
  <div
    class="flex flex-row w-full items-center px-1 py-1 group bg-charcoal-500 border-[1px] border-charcoal-500 {$$props.class ||
      ''}"
    class:hover:bg-charcoal-900="{!readonly && !disabled}"
    class:focus-within:bg-charcoal-900="{!readonly && !disabled}"
    class:hover:rounded-md="{!readonly && !disabled}"
    class:focus-within:rounded-md="{!readonly && !disabled}"
    class:border-b-purple-500="{!readonly && !disabled && !error}"
    class:border-b-red-500="{!readonly && !disabled && error}"
    class:hover:border-purple-400="{!readonly && !disabled && !error}"
    class:hover:border-red-400="{!readonly && !disabled && error}"
    class:focus-within:border-purple-500="{!readonly && !disabled && !error}"
    class:focus-within:border-red-500="{!readonly && !disabled && error}"
    class:border-b-charcoal-100="{readonly || disabled}">
    <slot name="left" />
    <input
      bind:this="{element}"
      on:input
      class="grow px-1 outline-0 bg-charcoal-500 text-sm placeholder:text-gray-700 overflow-hidden"
      class:text-white="{!disabled}"
      class:text-gray-700="{disabled}"
      class:group-hover:bg-charcoal-900="{!readonly && !disabled}"
      class:group-focus-within:bg-charcoal-900="{!readonly && !disabled}"
      class:group-hover-placeholder:text-gray-900="{!readonly && !disabled}"
      name="{name}"
      type="text"
      disabled="{disabled}"
      readonly="{readonly}"
      required="{required}"
      placeholder="{placeholder}"
      id="{id}"
      aria-label="{$$props['aria-label']}"
      aria-invalid="{$$props['aria-invalid']}"
      bind:value="{value}" />
    {#if error}
      <span class="px-1 text-red-500" aria-label="error">
        <Fa icon="{faCircleExclamation}" />
      </span>
    {/if}
    {#if clearable}
      <button
        class="px-1 cursor-pointer text-gray-700 group-hover:text-gray-900 group-focus-within:text-gray-900"
        class:hidden="{!value || readonly || disabled}"
        aria-label="clear"
        on:click="{onClear}">
        <Fa icon="{faXmark}" />
      </button>
    {/if}
    <slot name="right" />
  </div>
  {#if error && error.length > 0}
    <span class="text-sm text-red-500">{error}</span>
  {/if}
</div>
