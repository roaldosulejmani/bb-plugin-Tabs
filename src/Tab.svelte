<script>
  import { getContext, onMount } from "svelte"

  let {
    tab, // object of type { id: number, name: text, icon: text }
    iconSize,
    iconsOnly,
    emphasized,
    emphasizedColor = "var(--spectrum-global-color-indigo-600)",
  } = $props()

  let isSelected = $state(false)
  let tabBox = $state()

  const tabStore = getContext("tabStore")

  function updateBoundingBox() {
    if ($tabStore.selectedTab === tab.id) {
      $tabStore.selectedTabBox = tabBox?.getBoundingClientRect()
      isSelected = true
    } else {
      isSelected = false
    }
  }

  const onClick = () => {
    $tabStore.selectedTab = tab.id
  }

  let tabColor = $derived(
    isSelected
      ? emphasized
        ? emphasizedColor
        : "var(--spectrum-global-color-gray-900)"
      : "var(--spectrum-global-color-gray-600)"
  )

  $effect(() => {
    updateBoundingBox($tabStore.selectedTab, $tabStore.direction, $tabStore.size)
  })

  onMount(() => updateBoundingBox())
</script>

<!-- svelte-ignore a11y_click_events_have_key_events -->
<!-- svelte-ignore a11y_no_static_element_interactions -->
<!-- svelte-ignore a11y_no_noninteractive_tabindex -->
<div
  bind:this={tabBox}
  onclick={onClick}
  class="spectrum-Tabs-item"
  tabindex="0"
  role="tab"
  style="color: {tabColor};"
>
  {#if tab.icon}
    <i class="{tab.icon} {iconSize}"></i>
  {/if}
  {#if !iconsOnly}
    <span class="spectrum-Tabs-itemLabel">{tab.name}</span>
  {/if}
</div>

<style>
  .spectrum-Tabs-item {
    display: flex;
    flex-direction: row;
    padding: 0px 4px;
    color: var(--spectrum-global-color-gray-600);
    align-items: center;
    gap: 8px;
  }
</style>
