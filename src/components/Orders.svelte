<script>
  import { supabase } from "../lib/supabase";
  import { onMount } from "svelte";
  import OrderModal from "./OrderModal.svelte";

  let orders = [];
  let selectedOrder = null;

  async function fetchOrders() {
    const { data: fetchedOrders, error } = await supabase
      .from("orders")
      .select("*, shop:shops(name)");
    if (error) {
      console.error("Failed to fetch orders", error);
    } else {
      console.log("Fetched orders successfully", fetchedOrders);
      orders = fetchedOrders;
    }
  }

  function openModal(order) {
    selectedOrder = order;
  }

  function closeModal() {
    selectedOrder = null;
  }

  onMount(() => {
    fetchOrders();
  });
</script>

<main class="p-4">
  <h1 class="text-2xl font-bold mb-6">Order Details</h1>

  <div class="grid grid-cols-1 gap-4">
    {#each orders as order}
      <div class="bg-white shadow-md rounded-lg p-4 group">
        <div class="flex flex-row justify-between items-center mb-2">
          <h2 class="text-xl font-bold">{order.shop?.name || "N/A"}</h2>
          <button
            class="bg-blue-500 hover:bg-blue-600 text-white font-bold py-1 px-2 rounded transition duration-300 opacity-0 group-hover:opacity-100"
            on:click={() => openModal(order)}
          >
            Details
          </button>
        </div>
        <p class="text-gray-600">
          <span class="font-semibold">Items:</span>
          {order.items}
        </p>
        <p class="text-gray-600">
          <span class="font-semibold">Order Date:</span>
          {order.order_date}
        </p>
        <p class="text-gray-600">
          <span class="font-semibold">Last Delivery Date:</span>
          {order.Last_Delivery_date}
        </p>
        <p class="text-gray-600">
          <span class="font-semibold">Periodic:</span>
          {order.Periodic ? "Yes" : "No"}
        </p>
        <p class="text-gray-600">
          <span class="font-semibold">Period:</span>
          {order.Period}
        </p>
      </div>
    {/each}
  </div>

  {#if selectedOrder}
    <OrderModal
      order={selectedOrder}
      on:close={closeModal}
      on:orderUpdated={fetchOrders}
      on:orderDeleted={fetchOrders}
    />
  {/if}
</main>