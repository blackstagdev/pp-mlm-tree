<script>
  import { onMount } from "svelte";
  import TreeNode from "./lib/TreeNode.svelte";

  let tree = [];
  let filteredTree = [];
  let originalTree = [];
  let loading = true;
  let error = null;
  let search = "";
  let expanded = false;

  function buildTree(data) {
    const map = {};
    data.forEach((d) => (map[d.id] = { ...d, children: [] }));

    data.forEach((d) => {
      if (d.parent && map[d.parent]) {
        map[d.parent].children.push(map[d.id]);
      }
    });

    return Object.values(map).filter((n) => !n.parent || !map[n.parent]);
  }

  function searchTree(query) {
    if (!query) {
      filteredTree = tree;
      return;
    }

    const lower = query.toLowerCase();
    const results = [];

    function dfs(node) {
      if (
        node.name?.toLowerCase().includes(lower) ||
        node.email?.toLowerCase().includes(lower)
      ) {
        return node; // return whole node (with its children)
      }
      // search in children
      const matchedChildren = node.children.map(dfs).filter(Boolean);

      if (matchedChildren.length) {
        return { ...node, children: matchedChildren };
      }
      return null;
    }

    for (const root of tree) {
      const match = dfs(root);
      if (match) results.push(match);
    }
    filteredTree = results;
    expanded = true;
  }

  $: searchTree(search); // reactive to input

  onMount(async () => {
    try {
      // 1. Fetch MLM tree
      const treeRes = await fetch(
        "https://api.goaffpro.com/v1/admin/mlm/tree",
        {
          headers: {
            "X-GOAFFPRO-ACCESS-TOKEN":
              "04fe42fda80a9b50f064c6314fdc8c4db84cd779c1988250000b4e91e8a273bd",
          },
        }
      );
      if (!treeRes.ok) throw new Error(await treeRes.text());
      const treeData = await treeRes.json();
      const arr = Array.isArray(treeData) ? treeData : treeData.tree;

      // 2. Fetch affiliates
      const affRes = await fetch(
        "https://api.goaffpro.com/v1/admin/affiliates?fields=id,name,email,total_referral_earnings,total_network_earnings,total_other_earnings,status",
        {
          headers: {
            "X-GOAFFPRO-ACCESS-TOKEN":
              "04fe42fda80a9b50f064c6314fdc8c4db84cd779c1988250000b4e91e8a273bd",
          },
        }
      );
      if (!affRes.ok) throw new Error(await affRes.text());
      const affData = await affRes.json();
      const affiliates = Array.isArray(affData) ? affData : affData.affiliates;

      const affMap = {};
      affiliates.forEach((a) => (affMap[a.id] = a));

      const merged = arr.map((n) => ({
        ...n,
        ...affMap[n.id],
      }));

      tree = buildTree(merged);
      filteredTree = tree;
      originalTree = tree;
    } catch (e) {
      error = e.message;
    } finally {
      loading = false;
    }
  });
</script>

{#if loading}
  <p>Loading GoAffPro network...</p>
{:else if error}
  <p style="color:red">Error: {error}</p>
{:else}
  <div>
    <input
      type="text"
      placeholder="Search affiliates by name or email..."
      bind:value={search}
      style="margin-bottom:10px; padding:5px; width:300px"
    />
    <ul style="list-style: none;">
      {#each filteredTree as root (root.id)}
        <TreeNode node={root} searchExpand={expanded} />
      {/each}
      {#if filteredTree.length === 0}
        <p>No affiliates found</p>
      {/if}
    </ul>
  </div>
{/if}
