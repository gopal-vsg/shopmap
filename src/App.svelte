<script lang="ts">
  import Shops from "./components/Shops.svelte";
  import Orders from "./components/Orders.svelte";
  import Leaflet from "./components/Leaflet.svelte";
  import PlaceOrder from "./components/PlaceOrder.svelte";
  import CreateShop from "./components/CreateShop.svelte";
  let selectedOption = "hello";
</script>

<main class="min-h-screen bg-gray-100">
  <nav class="bg-blue-500 p-4 shadow-md sp flex justify-between items-center">
    <!-- svelte-ignore a11y-invalid-attribute -->
    <!-- svelte-ignore a11y-missing-attribute -->
    <a class="text-white text-2xl font-bold">Shop Map</a>
    <div class="flex space-x-4">
      <button class="btn" onclick="PlaceOrder.showModal()">Add shop</button>
      <button class="btn" onclick="PlaceShop.showModal()">Place order</button>
    </div>
    
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
  <!-- You can open the modal using ID.showModal() method -->

<dialog id="PlaceShop" class="modal">
  <div class="modal-box">
    <form method="dialog">
      <button class="btn btn-sm btn-circle btn-ghost absolute right-2 top-2">✕</button>
    </form>
    <PlaceOrder/>
  </div>
  <form method="dialog" class="modal-backdrop">
    <button>close</button>
  </form>
</dialog>
<dialog id="PlaceOrder" class="modal">
  <div class="modal-box">
    <form method="dialog">
      <button class="btn btn-sm btn-circle btn-ghost absolute right-2 top-2">✕</button>
    </form>
    <CreateShop/>
  </div>
  <form method="dialog" class="modal-backdrop">
    <button>close</button>
  </form>
</dialog>
</main>

<style>
  /* You can add any global styles here if needed */
</style>