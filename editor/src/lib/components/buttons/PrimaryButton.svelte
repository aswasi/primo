<script>
  import { createEventDispatcher } from 'svelte';
  import Icon from '@iconify/svelte'
  const dispatch = createEventDispatcher();

  export let id = null;
  export let variants = '';
  export let type = 'button';
  export let disabled = false;
  export let icon = null;
  export let label = ''

</script>

<button
  {id}
  class="primary-button {variants}"
  {disabled}
  on:click={(e) => dispatch('click', e)}
  {type}>
  {#if icon}
    <Icon {icon} />
  {/if}
  <slot>
    <span>{label}</span>
  </slot>
</button>

<style lang="postcss">
  .primary-button {
    width: 100%;
    background: var(--PrimaryButton-bg, var(--primo-color-brand));
    color: var(--PrimaryButton-color, var(--primo-color-black));
    padding: 0.5rem;
    margin-top: var(--PrimaryButton-mt, 0);
    margin-bottom: var(--PrimaryButton-mb, 0);
    margin-left: var(--PrimaryButton-ml, 0);
    margin-right: var(--PrimaryButton-mr, 0);

    border-top-right-radius: var(--PrimaryButton-round-tr, 0.25rem);
    border-top-left-radius: var(--PrimaryButton-round-tl, 0.25rem);
    border-bottom-right-radius: var(--PrimaryButton-round-br, 0.25rem);
    border-bottom-left-radius: var(--PrimaryButton-round-bl, 0.25rem);
    transition: color 0.1s, background-color 0.1s;

    display: flex;
    justify-content: center;
    align-items: center;
    text-align: center;
    gap: 0.5rem;
    font-weight: 500;

    &:hover {

      box-shadow: var(--primo-ring-primogreen-thick);
      /* background: var(
        --PrimaryButton-bg-hover,
        var(--primo-color-brand-dark)
      );
      color: var(--PrimaryButton-color-hover, var(--primo-color-white)); */
    }

    &[disabled] {
      background: var(--color-gray-6);
      cursor: not-allowed;
    }
  }

</style>
