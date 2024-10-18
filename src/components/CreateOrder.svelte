<script>
  import { supabase } from "../lib/supabase";
  import { onMount } from "svelte";

  let shops = [];
  let selectedShopId = "";
  let items = "";
  let periodic = false;
  let period = 0;
  let orderValue = 0;
  let message = "";
  let myModal1;

  onMount(async () => {
    const { data, error } = await supabase.from("retailers").select("id, name");
    if (error) {
      console.error("Error getting details of shops:", error);
    } else {
      shops = data;
    }
  });

  async function createOrder() {
    if (!selectedShopId || !items) {
      message = "Please select a shop and enter items.";
      showModal();
      return;
    }

    if (periodic && period <= 0) {
      message = "Please enter a valid period.";
      showModal();
      return;
    }

    if (orderValue <= 0) {
      message = "Please enter a valid order value.";
      showModal();
      return;
    }

    const order = {
      shop_id: selectedShopId,
      items: items,
      order_date: new Date().toISOString(),
      Periodic: periodic,
      Period: periodic ? period : null,
      order_value: orderValue,
    };

    const { error } = await supabase.from("orders").insert([order]);

    if (error) {
      console.error("Error placing order:", error);
      message = "Failed to create order. Please try again.";
    } else {
      message = "Order created successfully!";
      items = "";
      selectedShopId = "";
      periodic = false;
      period = 0;
      orderValue = 0;
      location.reload();
    }
    showModal();
  }

  function showModal() {
    if (myModal1) {
      myModal1.showModal();
    }
  }
</script>

<main class="p-4 max-w-md mx-auto">
  <h1 class="text-2xl font-bold mb-4">Create Order</h1>

  <div class="mb-4">
    <label for="shop-select" class="block mb-2">Select a Retailer:</label>
    <select
      id="shop-select"
      class="select select-bordered w-full"
      bind:value={selectedShopId}
    >
      <option value="">Choose a Retailer</option>
      {#each shops as shop}
        <option value={shop.id}>{shop.name}</option>
      {/each}
    </select>
  </div>

  <div class="mb-4">
    <label for="items-input" class="block mb-2">Enter items:</label>
    <textarea
      id="items-input"
      class="border p-2 w-full"
      placeholder="Enter items separated by commas"
      bind:value={items}
    ></textarea>
  </div>

  <div class="mb-4 flex items-center">
    <label for="periodic" class="mr-2">Periodic</label>
    <input type="checkbox" id="periodic" bind:checked={periodic}>
  </div>

  {#if periodic}
    <div class="mb-4">
      <label for="period">Period:</label>
      <input 
        type="number"
        id="period"
        class="border p-2 w-full"
        min="1"
        bind:value={period}
      >
    </div>
  {/if}

  <div class="mb-4">
    <label for="order-value" class="block mb-2">Order Value (Rupees):</label>
    <input 
      type="number"
      id="order-value"
      class="border p-2 w-full"
      bind:value={orderValue}
      min="0"
    >
  </div>

  <button 
    class="p-3 rounded-md w-full bg-purple-500 text-white" 
    on:click={createOrder}
  >
    Create Order
  </button>

  <dialog bind:this={myModal1} class="modal">
    <div class="modal-box">
      <h3 class="text-lg font-bold">{message}</h3>
    </div>
    <form method="dialog" class="modal-backdrop">
      <button type="button" on:click={() => myModal1.close()}>Close</button>
    </form>
  </dialog>
</main>