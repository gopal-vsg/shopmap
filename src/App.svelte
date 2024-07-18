<script lang="ts">
  import Shops from "./components/Shops.svelte";
  import Orders from "./components/Orders.svelte";
  import Leaflet from "./components/Leaflet.svelte";
  let selectedOption = "hello";
</script>

<main class="min-h-screen bg-gray-100">
  <nav class="bg-blue-500 p-4 shadow-md">
    <!-- svelte-ignore a11y-invalid-attribute -->
    <!-- svelte-ignore a11y-missing-attribute -->
    <a class="text-white text-2xl font-bold">Shop Map</a>
  </nav>

  <Leaflet />
  
  <div class="container mx-auto mt-8">
    <div class="w-3/4 mx-auto">
      <select 
        bind:value={selectedOption}
        class="w-24 p-2 mb-4 border border-gray-300 rounded-md shadow-sm focus:border-blue-500 focus:ring focus:ring-blue-200 focus:ring-opacity-50"
      >
        <option value="hello">Shops</option>
        <option value="world">Orders</option>
      </select>
      
      <div class="bg-white rounded-lg shadow-md p-6">
        {#if selectedOption === "hello"}
          <Shops />
        {:else}
          <Orders />
        {/if}
      </div>
    </div>
  </div>
</main>

<style>
  /* You can add any global styles here if needed */
</style>