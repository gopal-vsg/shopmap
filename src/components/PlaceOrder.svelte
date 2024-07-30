<script>
    import { supabase } from "../lib/supabase";
    import { onMount } from "svelte";
  
    let shops = [];
    let selectedShopId = null;
    let items = "";
    let message = "";
    let myModal1;
  
    onMount(async () => {
      const { data, error } = await supabase.from("shops").select("id, name");
      if (error) {
        console.error("Error getting details of shops:", error);
      } else {
        console.log("Shops data fetched successfully");
        shops = data;
      }
    });
  
    async function placeOrder() {
      if (!selectedShopId || !items) {
        message = "Please select a shop and enter items.";
        myModal1.showModal();
        return;
      }
  
      const { data, error } = await supabase.from("orders").insert([
        {
          shop_id: selectedShopId,
          items: items,
          order_date: new Date().toISOString(),
          delivered: false,
        },
      ]);
  
      if (error) {
        console.error("Error placing order:", error);
        message = "Failed to place order. Please try again.";
      } else {
        console.log("Order placed successfully");
        message = "Order placed successfully!";
        items = "";
        selectedShopId = null;
        location.reload();
      }
      myModal1.showModal();
    }
  </script>
  
  <main class="p-4 max-w-md mx-auto">
    <h1 class="text-2xl font-bold mb-4">Place your order</h1>
  
    <div class="mb-4">
      <label for="shop-select" class="block mb-2">Select a shop:</label>
      <select
        id="shop-select"
        class="select select-bordered w-full"
        bind:value={selectedShopId}
      >
        <option value={null}>Choose a shop</option>
        {#each shops as shop}
          <option value={shop.id}>{shop.name}</option>
        {/each}
      </select>
    </div>
  
    <div class="mb-4">
      <label for="items-input" class="block mb-2">Enter items:</label>
      <textarea
        id="items-input"
        class=" border p-2 w-full"
        placeholder="Enter items"
        bind:value={items}
      ></textarea>
    </div>
  
    <button class="p-3 rounded-md w-full bg-purple-500 text-white" on:click={placeOrder}>
      Place Order
    </button>
    <dialog bind:this={myModal1} class="modal">
      <div class="modal-box ">
        <h3 class="text-lg font-bold">{message}</h3>
      </div>
      <form method="dialog" class="modal-backdrop">
        <button>close</button>
      </form>
    </dialog>
  </main>
  