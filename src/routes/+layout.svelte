<script lang="ts">
  import "../app.css";
  import favicon from "$lib/assets/favicon.svg";

  let { children } = $props();
</script>

<svelte:head>
  <title>Tic-Tac-Toe - Play Against AI</title>
  <link rel="icon" href={favicon} />
</svelte:head>

{@render children?.()}
