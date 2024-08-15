<script lang="ts">
  import Shops from "./components/Shops.svelte";
  import Orders from "./components/Orders.svelte";
  import Leaflet from "./components/Leaflet.svelte";
  import CreateOrder from "./components/CreateOrder.svelte";
  import CreateShop from "./components/CreateShop.svelte";
  import Login from "./components/Login.svelte";
  import { onMount } from "svelte";
  import { supabase } from "./lib/supabase";

  let selectedOption = "shops";
  let user = null;
  let shopMap;
  let loginDialog: HTMLDialogElement;
  let createShopDialog: HTMLDialogElement;
  let createOrderDialog: HTMLDialogElement;

  const onShopSelected = (event) => {
    const { lat, lng } = event.detail;
    if (shopMap) {
      shopMap.centerMap(lat, lng);
    }
  };

  onMount(async () => {
    const {
      data: { user: currentUser },
    } = await supabase.auth.getUser();
    user = currentUser;
    if (!user) {
      loginDialog.showModal();
    }
  });

  const signOut = async () => {
    await supabase.auth.signOut();
    user = null;
    location.reload();
    loginDialog.showModal();
  };
</script>

<main class="min-h-screen bg-gray-100">
  {#if user}
    <nav class="bg-blue-500 p-4 shadow-md">
      <div class="container mx-auto flex flex-wrap justify-between items-center">
        <!-- svelte-ignore a11y-missing-attribute -->
        <a class="text-white text-2xl font-bold mb-2 md:mb-0 w-full md:w-auto text-center md:text-left">Shop Map</a>
        <div class="flex flex-wrap justify-center md:justify-end w-full md:w-auto space-y-2 md:space-y-0 md:space-x-2">
          <button class="btn w-full sm:w-auto" on:click={() => createOrderDialog.showModal()}>New shop</button>
          <button class="btn w-full sm:w-auto" on:click={() => createShopDialog.showModal()}>Create order</button>
          <button class="btn w-full sm:w-auto" on:click={signOut}>Logout</button>
        </div>
      </div>
    </nav>

    <div class="container mx-auto p-4">
      <div class="flex flex-col md:flex-row gap-4">
        <div class="w-full md:w-2/5">
          <div class="bg-white rounded-lg shadow-md p-6">
            <div class="mb-4">
              <label for="view-select" class="block text-sm font-medium text-gray-700 mb-2">Select View:</label>
              <select
                id="view-select"
                bind:value={selectedOption}
                class="w-full p-2 border border-gray-300 rounded-md shadow-sm focus:border-blue-500 focus:ring focus:ring-blue-200 focus:ring-opacity-50"
              >
                <option value="shops">Shops</option>
                <option value="orders">Orders</option>
              </select>
            </div>
    
            <div class="overflow-auto max-h-[calc(100vh-200px)]">
              {#if selectedOption === "shops"}
                <Shops on:shopSelected={onShopSelected} />
              {:else}
                <Orders />
              {/if}
            </div>
          </div>
        </div>
    
        <div class="w-full md:w-3/5">
          <div class="bg-white rounded-lg shadow-md p-6 h-full">
            <Leaflet bind:this={shopMap} />
          </div>
        </div>
      </div>
    </div>

    <dialog bind:this={createShopDialog} class="modal">
      <div class="modal-box">
        <form method="dialog">
          <button class="btn btn-sm btn-circle btn-ghost absolute right-2 top-2">✕</button>
        </form>
        <CreateOrder />
      </div>
      <form method="dialog" class="modal-backdrop">
        <button>close</button>
      </form>
    </dialog>

    <dialog bind:this={createOrderDialog} class="modal">
      <div class="modal-box">
        <form method="dialog">
          <button class="btn btn-sm btn-circle btn-ghost absolute right-2 top-2">✕</button>
        </form>
        <CreateShop />
      </div>
      <form method="dialog" class="modal-backdrop">
        <button>close</button>
      </form>
    </dialog>
  {:else}
    <dialog bind:this={loginDialog} class="modal">
      <div>
        <Login />
      </div>
    </dialog>
  {/if}
</main>

<style>
  /* You can add any global styles here if needed */
</style>