<script>
// @ts-nocheck

  import { supabase } from "../lib/supabase";
  import { onMount, createEventDispatcher } from "svelte";

  const dispatch = createEventDispatcher();
  
  let data = [];
  async function fetchShops() {
    const { data: fetchedShops, error } = await supabase
      .from("shops")
      .select("*");

    if (error) {
      console.error("Error fetching data:", error);
    } else {
      console.log("Data fetched successfully:", fetchedShops);
      data = fetchedShops;
    }
  }

  function handleRowClick(lat, lng) {
    dispatch('shopSelected', { lat, lng });
  }

  function openModal(shopId) {
    const modal = document.getElementById(`shop_modal_${shopId}`);
    if (modal) {
      modal.showModal();
    }
  }

  onMount(() => {
    fetchShops();
  });
</script>

<main class="p-4">
  <h1 class="text-2xl font-bold mb-6">Shop Details</h1>
  
  <div class="grid grid-cols-1 gap-4">
    {#each data as shop}

      <div 
        class="bg-white shadow-md rounded-lg p-4 cursor-pointer hover:shadow-lg transition-shadow duration-300 group"
        on:click={() => handleRowClick(shop.lat, shop.lng)}
      >
        <div class="flex flex-row justify-between items-center mb-2">
          <h2 class="text-xl font-bold">{shop.name}</h2>
          <button 
            class="bg-blue-500 hover:bg-blue-600 text-white font-bold py-1 px-2 rounded transition duration-300 opacity-0 group-hover:opacity-100"
            on:click|stopPropagation={() => openModal(shop.id)}
          >
            Details
          </button>
        </div>
        <p class="text-gray-600">Owner: {shop.owner}</p>
        <p class="text-gray-600">Phone: {shop.phone}</p>
        <p class="text-gray-600">Address: {shop.address}</p>
      </div>

      <dialog id="shop_modal_{shop.id}" class="modal">
        <div class="modal-box">
          <h3 class="text-lg font-bold">{shop.name} Details</h3>
          <p class="py-2"><strong>Owner:</strong> {shop.owner}</p>
          <p class="py-2"><strong>Phone:</strong> {shop.phone}</p>
          <p class="py-2"><strong>Address:</strong> {shop.address}</p>
          <p class="py-2"><strong>Latitude:</strong> {shop.lat}</p>
          <p class="py-2"><strong>Longitude:</strong> {shop.lng}</p>
        </div>
        <form method="dialog" class="modal-backdrop">
          <button>close</button>
        </form>
      </dialog>
    {/each}
  </div>
</main>