<script>
  export let node;
  let expanded = false;

  function toggle(ev) {
    ev.stopPropagation();
    expanded = !expanded;
  }

  const formatterUSD = new Intl.NumberFormat("en-US", {
    style: "currency",
    currency: "USD",
  });
</script>

<li>
  <div class="node" on:click={toggle}>
    <span class="toggle">
      {#if node.children && node.children.length}
        {expanded ? "-" : "+"}
      {:else}
        -
      {/if}
    </span>
    <span
      >{node.name}
      <small>{formatterUSD.format(node.subtotal_revenue)}</small></span
    >
  </div>

  {#if expanded && node.children && node.children.length}
    <ul>
      {#each node.children as child (child.id)}
        <!-- recursive render of the same component -->
        <svelte:self node={child} />
      {/each}
    </ul>
  {/if}
</li>

<style>
  ul {
    list-style: none;
    padding-left: 1rem;
    margin: 0;
  }
  li {
    list-style: none;
    margin: 4px 0;
  }
  .node {
    display: flex;
    align-items: center;
    cursor: pointer;
    user-select: none;
  }
  .toggle {
    width: 1.2em;
    display: inline-block;
    text-align: center;
  }
</style>
