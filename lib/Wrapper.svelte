<script>
  import Component from "../src/Component.svelte"
  import Boundary from "./Boundary.js"

  let { children, ...restProps } = $props()
</script>

<Boundary>
  <Component {...restProps}>
    {#snippet children()}
      {@render children?.()}
    {/snippet}
  </Component>
</Boundary>
