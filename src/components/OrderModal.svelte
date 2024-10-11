<script>
  import { createEventDispatcher } from "svelte";
  import { supabase } from "../lib/supabase";

  export let order;

  const dispatch = createEventDispatcher();

  let editMode = false;
  let editedOrder = { ...order, shop_name: order.shop?.name || "N/A" };

  function closeModal() {
    dispatch("close");
  }

  async function saveChanges() {
    // Prepare the update object, excluding the shop property
    const updateData = {
      items: editedOrder.items,
      order_date: editedOrder.order_date,
      Last_Delivery_date: editedOrder.Last_Delivery_date,
      Periodic: editedOrder.Periodic,
      Period: editedOrder.Period,
    };

    const { data, error } = await supabase
      .from("orders")
      .update(updateData)
      .eq("id", order.id);

    if (error) {
      console.error("Failed to update order", error);
    } else {
      console.log("Order updated successfully", data);

      // Update the shop name if it has changed
      if (editedOrder.shop_name !== order.shop?.name) {
        const { data: shopData, error: shopError } = await supabase
          .from("retailers")
          .update({ name: editedOrder.shop_name })
          .eq("id", order.shop?.id);

        if (shopError) {
          console.error("Failed to update shop name", shopError);
        } else {
          console.log("Shop name updated successfully", shopData);
        }
      }

      dispatch("orderUpdated");
      editMode = false;
    }
  }

  async function markAsDelivered() {
    const { data, error } = await supabase
      .from("orders")
      .update({ Last_Delivery_date: new Date().toISOString() })
      .eq("id", order.id);

    if (error) {
      console.error("Failed to mark order as delivered", error);
    } else {
      console.log("Order marked as delivered", data);
      dispatch("orderUpdated");
    }
  }

  async function deleteOrder() {
    if (confirm("Are you sure you want to delete this order?")) {
      const { data, error } = await supabase
        .from("orders")
        .delete()
        .eq("id", order.id);

      if (error) {
        console.error("Failed to delete order", error);
      } else {
        console.log("Order deleted successfully", data);
        dispatch("orderDeleted");
        closeModal();
      }
    }
  }
</script>

<div class="modal modal-open">
  <div class="modal-box">
    <h3 class="font-bold text-lg">Order Details</h3>
    {#if editMode}
      <div class="form-control">
        <label class="label">
          <span class="label-text">Shop</span>
        </label>
        <input
          type="text"
          bind:value={editedOrder.shop_name}
          class="input input-bordered"
        />

        <label class="label">
          <span class="label-text">Items</span>
        </label>
        <input
          type="text"
          bind:value={editedOrder.items}
          class="input input-bordered"
        />

        <label class="label">
          <span class="label-text">Order Date</span>
        </label>
        <input
          type="date"
          bind:value={editedOrder.order_date}
          class="input input-bordered"
        />

        <label class="label">
          <span class="label-text">Last Delivery Date</span>
        </label>
        <input
          type="date"
          bind:value={editedOrder.Last_Delivery_date}
          class="input input-bordered"
        />

        <label class="label">
          <span class="label-text">Periodic</span>
        </label>
        <input
          type="checkbox"
          bind:checked={editedOrder.Periodic}
          class="checkbox"
        />

        <label class="label">
          <span class="label-text">Period</span>
        </label>
        <input
          type="text"
          bind:value={editedOrder.Period}
          class="input input-bordered"
        />
      </div>
    {:else}
      <div class="p-4 bg-white rounded-lg shadow-md">
        <div class="space-y-2">
          <p class="text-gray-700">
            <span class="font-medium">Shop:</span>
            {order.shop?.name || "N/A"}
          </p>
          <p class="text-gray-700">
            <span class="font-medium">Items:</span>
            {order.items}
          </p>
          <p class="text-gray-700">
            <span class="font-medium">Order Date:</span>
            {order.order_date}
          </p>
          <p class="text-gray-700">
            <span class="font-medium">Last Delivery Date:</span>
            {order.Last_Delivery_date}
          </p>
          <p class="text-gray-700">
            <span class="font-medium">Periodic:</span>
            {order.Periodic ? "Yes" : "No"}
          </p>
          <p class="text-gray-700">
            <span class="font-medium">Period:</span>
            {order.Period}
          </p>
        </div>
      </div>
    {/if}

    <div class="flex justify-end space-x-2 mt-4">
        {#if editMode}
          <button 
            class="px-4 py-2 bg-blue-500 text-white rounded hover:bg-blue-600 transition duration-300"
            on:click={saveChanges}
          >
            Save
          </button>
          <button 
            class="px-4 py-2 bg-gray-300 text-gray-700 rounded hover:bg-gray-400 transition duration-300"
            on:click={() => (editMode = false)}
          >
            Cancel
          </button>
        {:else}
          <button 
            class="px-4 py-2 bg-blue-500 text-white rounded hover:bg-blue-600 transition duration-300"
            on:click={() => (editMode = true)}
          >
            Edit
          </button>
          <button 
            class="px-4 py-2 bg-green-500 text-white rounded hover:bg-green-600 transition duration-300"
            on:click={markAsDelivered}
          >
            Mark as Delivered
          </button>
          <button 
            class="px-4 py-2 bg-red-500 text-white rounded hover:bg-red-600 transition duration-300"
            on:click={deleteOrder}
          >
            Delete
          </button>
          <button 
            class="px-4 py-2 bg-gray-300 text-gray-700 rounded hover:bg-gray-400 transition duration-300"
            on:click={closeModal}
          >
            Close
          </button>
        {/if}
      </div>
  </div>
</div>
