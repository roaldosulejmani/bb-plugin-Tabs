<script>
  import "@spectrum-css/tabs/dist/index-vars.css"
  import { getContext, onMount, setContext } from "svelte"
  import { writable } from "svelte/store"
  import Tab from "./Tab.svelte"

  let {
    size = "M",
    direction,
    centered = false,
    vertical = false,
    quiet = false,
    compact = false,
    emphasized = false,
    iconsOnly = false,
    emphasizedColor = "var(--spectrum-global-color-indigo-600)",
    children,
  } = $props()

  // Internal
  let container = $state()
  let iconSize = $state("ri-1x")
  let top = $state()
  let left = $state()
  let width = $state()
  let height = $state()

  let styles = $derived({
    pos_justify: centered ? "center" : "flex-start",
  })

  let cssVarStyles = $derived(
    Object.entries(styles)
      .map(([key, value]) => `--${key}:${value}`)
      .join(";")
  )

  let resolvedEmphasizedColor = $derived(
    emphasized
      ? emphasizedColor
      : "var(--spectrum-global-color-gray-600"
  )

  $effect(() => {
    switch (size) {
      case "S":
        iconSize = "ri-sm"
        break
      case "M":
        iconSize = "ri-1x"
        break
      case "L":
        iconSize = "ri-lg"
        break
      case "XL":
        iconSize = "ri-xl"
        break
    }
  })

  const { styleable } = getContext("sdk")
  const component = getContext("component")

  const tabStore = writable({
    tabs: [],
    selectedTab: 0,
    selectedTabBox: {},
    direction: "horizontal",
    centered: false,
    registerTab: function (id, name, icon) {
      this.tabs.push({ id: id, name: name, icon: icon })
      this.selectedTab = id
    },
    unregisterTab: function (id) {
      var indx = this.tabs.findIndex(v => v.id === id)
      this.tabs.splice(indx, indx >= 0 ? 1 : 0)
    },
    updateTab: function (id, name, icon) {
      var indx = this.tabs.findIndex(v => v.id === id)
      if (indx > -1) {
        this.tabs[indx].name = name
        this.tabs[indx].icon = icon
      }
    },
  })
  setContext("tabStore", tabStore)

  $effect(() => {
    $tabStore.direction = direction
  })
  $effect(() => {
    $tabStore.centered = centered
  })
  $effect(() => {
    $tabStore.size = size
  })
  $effect(() => {
    calculateIndicator($tabStore.selectedTab, $tabStore.direction)
  })

  function setActiveTab() {
    if ($tabStore.tabs.length > 0) $tabStore.selectedTab = $tabStore.tabs[0].id
  }

  function calculateIndicator() {
    let tabBox = $tabStore?.selectedTabBox
    if (tabBox) {
      if (direction === "horizontal") {
        width = tabBox?.width + "px"
        height = "2px"
        left = tabBox?.left - container?.getBoundingClientRect().left + "px"
        top = tabBox?.height + "px"
      } else {
        height = tabBox?.height + "px"
        width = "2px"
        top = tabBox?.top - container?.getBoundingClientRect().top + "px"
        left =
          tabBox?.left - container?.getBoundingClientRect().left - 8 + "px"
      }
    }
  }

  onMount(() => setActiveTab())
</script>

<div
  use:styleable={$component.styles}
  class="outer_container"
  class:vertical={direction === "vertical"}
>
  {#if $tabStore.tabs.length > 0}
    <div
      bind:this={container}
      class:spectrum-Tabs--quiet={quiet}
      class:spectrum-Tabs--vertical={direction === "vertical"}
      class:spectrum-Tabs--horizontal={direction !== "vertical"}
      class:spectrum-Tabs--compact={compact}
      class="spectrum-Tabs spectrum-Tabs--size{size}"
      style={cssVarStyles}
    >
      {#each $tabStore.tabs as tab}
        <Tab {tab} {iconsOnly} {iconSize} {emphasized} emphasizedColor={resolvedEmphasizedColor} />
      {/each}

      <div
        class="spectrum-Tabs-selectionIndicator"
        style="background-color: {resolvedEmphasizedColor}; width: {width}; height: {height}; left: {left}; top: {top};"
      ></div>
    </div>
  {:else}
    <div>Please add some Tab Containers to get started</div>
  {/if}
  <div class="spectrum-Tabs-content">
    {@render children?.()}
  </div>
</div>

<style>
  .outer_container {
    display: flex;
    flex-direction: column;
    gap: 1rem;
  }

  .vertical {
    flex-direction: row;
  }
  .spectrum-Tabs--quiet {
    display: flex;
    border-color: transparent !important;
    justify-content: var(--pos_justify) !important;
  }
  .spectrum-Tabs {
    position: relative;
    border-bottom-color: var(--spectrum-global-color-gray-200);
    justify-content: var(--pos_justify);
  }
  .spectrum-Tabs-content {
    flex-grow: 1;
  }
  .spectrum-Tabs-selectionIndicator {
    transition: all 200ms;
    background-color: var(--spectrum-global-color-gray-900);
  }

  .spectrum-Tabs--vertical {
    border-color: var(--spectrum-global-color-gray-200);
  }
</style>
