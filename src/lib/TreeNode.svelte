<script>
  export let node;
  export let searchExpand = false;

  let expanded = false;

  $: if (searchExpand) expanded = true;

  function toggle(e) {
    e.stopPropagation();
    expanded = !expanded;
  }

  const formatterUSD = new Intl.NumberFormat("en-US", {
    style: "currency",
    currency: "USD",
  });
</script>

<li class="tree-node">
  <button
    class="node"
    class:node-hover={node.children?.length}
    on:click={toggle}
  >
    {#if node.children && node.children.length}
      {expanded ? "-" : "+"}
    {:else}
      -
    {/if}
    <span class="label">
      {node.name}
      <small>{formatterUSD.format(node.subtotal_revenue)}</small>
    </span>
  </button>

  {#if expanded && node.children?.length}
    <ul>
      {#each node.children as child (child.id)}
        <svelte:self node={child} {searchExpand} />
      {/each}
    </ul>
  {/if}
</li>

<style>
  ul {
    list-style: none;
    margin: 0;
    padding-left: 1.5rem;
    position: relative;
  }

  .tree-node {
    position: relative;
    padding-left: 1rem;
  }

  /* vertical line */
  .tree-node::before {
    content: "";
    position: absolute;
    top: 0;
    left: 0;
    width: 1px;
    height: 100%;
    background: #ccc;
  }

  /* horizontal connector */
  .tree-node::after {
    content: "";
    position: absolute;
    top: 1.1rem;
    left: 0;
    width: 1rem;
    height: 1px;
    background: #ccc;
  }

  /* remove line for last child */
  .tree-node:last-child::before {
    height: 1.1rem;
  }

  .node {
    cursor: pointer;
    padding: 6px 10px;
    border-radius: 4px;
    text-align: left;
    width: 100%;
    background: transparent;
    outline: none;
  }

  .node:hover {
    outline: none;
    border: 0;
  }

  .node-hover:hover {
    background: #f5f5f5;
    color: #242424;
    outline: none;
    border: 0;
  }

  .label small {
    margin-left: 6px;
    color: #666;
    font-size: 0.8em;
  }
</style>
