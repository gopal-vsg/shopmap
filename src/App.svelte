<script lang="ts">
  import Leaflet from "./lib/Leaflet.svelte";
  import { supabase } from "./lib/supabase";
  import { onMount } from "svelte";

  let data = [];
  let orders = [];

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
    fetchShops();
    fetchOrders();
  });
</script>

<main>
  <div class="navbar bg-blue-100">
    <a href="#" class="btn btn-ghost text-xl">Shop Map</a>
  </div>
  <div>
    <Leaflet/>
  </div>
  <div>
    <p class="m-10 text-2xl">Shop Details</p>
  </div>
  <div class="overflow-x-auto">
    <table class="table table-zebra w-full">
      <thead>
        <tr>
          <th>Name</th>
          <th>Owner</th>
          <th>Phone</th>
          <th>Address</th>
        </tr>
      </thead>
      <tbody>
        {#each data as row}
          <tr>
            <td>{row.name}</td>
            <td>{row.owner}</td>
            <td>{row.phone}</td>
            <td>{row.address}</td>
          </tr>
        {/each}
      </tbody>
    </table>
  </div>
  <div>
    <p class="m-10 text-2xl">Order Details</p>
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

<style>
</style>