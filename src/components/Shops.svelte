<script>
  // @ts-nocheck

  import { supabase } from "../lib/supabase";
  import { onMount, createEventDispatcher } from "svelte";

  const dispatch = createEventDispatcher();
  
  let data = [];
  async function fetchRetailers() {
    const { data: fetchedRetailers, error } = await supabase
      .from("retailers")  // Fetch data from 'retailers' table
      .select("*");

    if (error) {
      console.error("Error fetching data:", error);
    } else {
      console.log("Data fetched successfully:", fetchedRetailers);
      data = fetchedRetailers;  // Assign fetched data to the local variable
    }
  }

  function handleRowClick(lat, lng) {
    dispatch('shopSelected', { lat, lng });
  }

  function openModal(retailerId) {
    const modal = document.getElementById(`retailer_modal_${retailerId}`);
    if (modal) {
      modal.showModal();
    }
  }

  onMount(() => {
    fetchRetailers();
  });
</script>

<main class="p-4">
  <h1 class="text-2xl font-bold mb-6">Retailer Details</h1>
  
  <div class="grid grid-cols-1 gap-4">
    {#each data as retailer}
      <div 
        class="bg-white shadow-md rounded-lg p-4 cursor-pointer hover:shadow-lg transition-shadow duration-300 group"
        on:click={() => handleRowClick(retailer.lat, retailer.lng)}
      >
        <div class="flex flex-row justify-between items-center mb-2">
          <h2 class="text-xl font-bold">{retailer.name}</h2>
          <button 
            class="bg-blue-500 hover:bg-blue-600 text-white font-bold py-1 px-2 rounded transition duration-300 opacity-0 group-hover:opacity-100"
            on:click|stopPropagation={() => openModal(retailer.id)}
          >
            Details
          </button>
        </div>
        <p class="text-gray-600">Owner: {retailer.owner}</p>
        <p class="text-gray-600">Phone: {retailer.phone}</p>
        <p class="text-gray-600">Address: {retailer.address}</p>
      </div>

      <dialog id="retailer_modal_{retailer.id}" class="modal">
        <div class="modal-box">
          <h3 class="text-lg font-bold">{retailer.name} Details</h3>
          <p class="py-2"><strong>Owner:</strong> {retailer.owner}</p>
          <p class="py-2"><strong>Phone:</strong> {retailer.phone}</p>
          <p class="py-2"><strong>Address:</strong> {retailer.address}</p>
          <p class="py-2"><strong>Latitude:</strong> {retailer.lat}</p>
          <p class="py-2"><strong>Longitude:</strong> {retailer.lng}</p>

          <!-- Display the notes -->
          {#if retailer.notes}
            <p class="py-2"><strong>Notes:</strong> {retailer.notes}</p>
          {/if}

          <!-- Display the audio file if it exists -->
          {#if retailer.audio_url}
            <p class="py-2"><strong>Audio:</strong></p>
            <audio controls>
              <source src={retailer.audio_url} type="audio/mpeg" />
              Your browser does not support the audio element.
            </audio>
          {/if}
        </div>
        <form method="dialog" class="modal-backdrop">
          <button>close</button>
        </form>
      </dialog>
    {/each}
  </div>
</main>
