<script>
  import { supabase } from "../lib/supabase";
  import { onMount } from "svelte";
  import OrderModal from "./OrderModal.svelte"; // Import the new component
  
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
  
  <main>
    <div>
      <p class="m-4 text-2xl">Order Details</p>
    </div>
    <div class="overflow-x-auto">
      <table class="table table-zebra w-full">
        <thead>
          <tr>
            <th>Shop</th>
            <th>Items</th>
            <th>Order Date</th>
            <th>Last Delivery Date</th>
            <th>Periodic</th>
            <th>Period</th>
          </tr>
        </thead>
        <tbody>
          {#each orders as order}
            <tr on:click={() => openModal(order)} class=" cursor-pointer">
              <td>{order.shop?.name || 'N/A'}</td>
              <td>{order.items}</td>
              <td>{order.order_date}</td>
              <td>{order.Last_Delivery_date}</td>
              <td>{order.Periodic ? 'Yes' : 'No'}</td>
              <td>{order.Period}</td>
            </tr>
          {/each}
        </tbody>
      </table>
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