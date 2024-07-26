<!--App.svelte-->
<script lang="ts">
  import Shops from "./components/Shops.svelte";
  import Orders from "./components/Orders.svelte";
  import Leaflet from "./components/Leaflet.svelte";
  import PlaceOrder from "./components/PlaceOrder.svelte";
  import CreateShop from "./components/CreateShop.svelte";
  import Login from "./components/Login.svelte";
  import { onMount } from "svelte";
  import { supabase } from "./lib/supabase";
  let selectedOption = "shops";
  let user = null;

  onMount(async()=>{
    const { data : { user: currentuser }} = await supabase.auth.getUser();
    user = currentuser;
    if(!user){
      const loginDialog = document.getElementById('loginDialog') as HTMLDialogElement;
      loginDialog.showModal();
    }
  })

  async function signOut() {
    await supabase.auth.signOut();
    user = null;
    const loginDialog = document.getElementById('loginDialog') as HTMLDialogElement;
    location.reload()
    loginDialog.showModal();
    
  }

</script>


<main class="min-h-screen bg-gray-100">
  {#if user}
  <nav class="bg-blue-500 p-4 shadow-md">
    <div class="container mx-auto flex flex-wrap justify-between items-center">
      <!-- svelte-ignore a11y-missing-attribute -->
      <a class="text-white text-2xl font-bold mb-2 md:mb-0 w-full md:w-auto text-center md:text-left">Shop Map</a>
      <div class="flex flex-wrap justify-center md:justify-end w-full md:w-auto space-y-2 md:space-y-0 md:space-x-2">
        <button class="btn w-full sm:w-auto" onclick="PlaceOrder.showModal()">New shop</button>
        <button class="btn w-full sm:w-auto" onclick="PlaceShop.showModal()">Place order</button>
        <button class="btn w-full sm:w-auto" on:click={signOut}>Logout</button>
      </div>
    </div>
  </nav>

    <Leaflet />
    
    <div class="container mx-auto mt-8">
      <div class="w-3/4 mx-auto">
        <select 
          bind:value={selectedOption}
          class="w-24 p-2 mb-4 border border-gray-300 rounded-md shadow-sm focus:border-blue-500 focus:ring focus:ring-blue-200 focus:ring-opacity-50"
        >
          <option value="shops">Shops</option>
          <option value="orders">Orders</option>
        </select>
        
        <div class="bg-white rounded-lg shadow-md p-6">
          {#if selectedOption === "shops"}
            <Shops />
          {:else}
            <Orders />
          {/if}
        </div>
      </div>
    </div>

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
  {:else}
    <dialog id="loginDialog" class="modal">
      <div>
        <Login/>
      </div>
    </dialog>
  {/if}
</main>

<style>
  /* You can add any global styles here if needed */
</style>