<script>
  import { supabase } from "../lib/supabase";
  import { onMount } from "svelte";

  let data = [];
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
  onMount(() => {
    fetchShops();
  });
</script>

<main>
  <div>
    <p class="m-4 text-2xl">Shop Details</p>
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
</main>
