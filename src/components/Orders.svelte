<script>
  import { supabase } from "../lib/supabase";
  import { onMount } from "svelte";

  let orders = [];

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
              <th>Delivery Date</th>
              <th>Delivered</th>
            </tr>
          </thead>
          <tbody>
            {#each orders as order}
              <tr>
                <td>{order.shop?.name || 'N/A'}</td>
                <td>{order.items}</td>
                <td>{order.order_date}</td>
                <td>{order.delivery_date}</td>
                <td>{order.delivered ? 'Yes' : 'No'}</td>
              </tr>
            {/each}
          </tbody>
        </table>
      </div>
</main>
