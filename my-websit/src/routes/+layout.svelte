
<script>
  import Sidebar from '#lib/components/Sidebar.svelte';

  let { children } = $props();
</script>

<div class="website">
  <Sidebar />

  <main>
    {@render children()}
  </main>
</div>

<style>
  :global(*) {
    box-sizing: border-box;
  }

  :global(html) {
    font-family: 'Geo', sans-serif;
  }

  :global(body) {
    margin: 0;
    background: #000;
    color: #f3f3ff;
  }

  .website {
    display: flex;
    width: 100%;
    height: 100vh;
    overflow: hidden;
    background-color: #000;
    background-image: linear-gradient(
      to bottom,
      transparent 0 6px,
      rgb(255 255 255 / 6%) 6px 7px
    );
    background-size: 100% 7px;
    background-repeat: repeat;
  }

  main {
    flex: 1;
    min-width: 0;
    height: 100%;
    background: transparent;
    color: #f3f3ff;
    display: flex;
    justify-content: center;
    align-items: center;
    overflow-x: hidden;
    overflow-y: auto;
  }
</style>
