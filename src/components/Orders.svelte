<script>
  import { supabase } from "../lib/supabase";
  import { onMount } from "svelte";
  import OrderModal from "./OrderModal.svelte";

  let orders = [];
  let selectedOrder = null;

  async function fetchOrders() {
    const { data: fetchedOrders, error } = await supabase
      .from("orders")
      .select("*, shop:retailers(name)");
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
          <div class="flex items-center space-x-2">
            <span class={order.Last_Delivery_date ? "text-green-600" : "text-yellow-600"}>
              {order.Last_Delivery_date ? "Delivered" : "Pending"}
            </span>
            <button
              class="bg-blue-500 hover:bg-blue-600 text-white font-bold py-1 px-2 rounded transition duration-300 opacity-0 group-hover:opacity-100"
              on:click={() => openModal(order)}
            >
              Edit Details
            </button>
          </div>
        </div>
        <p class="text-gray-600">
          <span class="font-semibold">Items:</span>
          {order.items}
        </p>
        <p class="text-gray-600">
          <span class="font-semibold">Order Value:</span>
          ₹{order.order_value?.toLocaleString() || "0"}
        </p>
        <p class="text-gray-600">
          <span class="font-semibold">Order Date:</span>
          {order.order_date}
        </p>
        {#if order.Last_Delivery_date}
          <p class="text-gray-600">
            <span class="font-semibold">Delivery Date:</span>
            {order.Last_Delivery_date}
          </p>
        {/if}
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