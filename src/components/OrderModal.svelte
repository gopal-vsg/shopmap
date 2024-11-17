<script>
  import { createEventDispatcher } from "svelte";
  import { supabase } from "../lib/supabase";

  export let order;

  const dispatch = createEventDispatcher();

  // Initialize in edit mode directly with the order details (removed periodic)
  let editedOrder = { 
    ...order, 
    shop_name: order.shop_id?.name || "N/A",
    status: order.Last_Delivery_date ? "Delivered" : "Pending",
    delivery_date: order.Last_Delivery_date || new Date().toISOString().split('T')[0]
  };

  function closeModal() {
    dispatch("close");
  }

  async function saveChanges() {
    const updateData = {
      items: editedOrder.items,
      order_date: editedOrder.order_date,
      Last_Delivery_date: editedOrder.status === "Delivered" ? editedOrder.delivery_date : null,
      order_value: editedOrder.order_value,
    };

    const { data, error } = await supabase
      .from("orders")
      .update(updateData)
      .eq("id", order.id);

    if (error) {
      console.error("Failed to update order", error);
    } else {
      console.log("Order updated successfully", data);
      
      if (editedOrder.shop_name !== order.shop?.name) {
        const { data: shopData, error: shopError } = await supabase
          .from("retailers")
          .update({ name: editedOrder.shop_name })
          .eq("id", order.shop_id?.id);

        if (shopError) {
          console.error("Failed to update shop name", shopError);
        }
      }

      dispatch("orderUpdated");
      closeModal();
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
    <h3 class="font-bold text-lg mb-4">Edit Order Details</h3>
    
    <div class="form-control space-y-4">
      <div>
        <label class="label">
          <span class="label-text">Shop Name</span>
        </label>
        <input
          type="text"
          bind:value={editedOrder.shop_name}
          class="input input-bordered w-full"
        />
      </div>

      <div>
        <label class="label">
          <span class="label-text">Items</span>
        </label>
        <input
          type="text"
          bind:value={editedOrder.items}
          class="input input-bordered w-full"
        />
      </div>

      <div>
        <label class="label">
          <span class="label-text">Order Value (₹)</span>
        </label>
        <input
          type="number"
          bind:value={editedOrder.order_value}
          class="input input-bordered w-full"
          min="0"
        />
      </div>

      <div>
        <label class="label">
          <span class="label-text">Order Date</span>
        </label>
        <input
          type="date"
          bind:value={editedOrder.order_date}
          class="input input-bordered w-full"
        />
      </div>

      <div>
        <label class="label">
          <span class="label-text">Delivery Status</span>
        </label>
        <select
          bind:value={editedOrder.status}
          class="select select-bordered w-full"
        >
          <option value="Pending">Pending</option>
          <option value="Delivered">Delivered</option>
        </select>
      </div>

      {#if editedOrder.status === "Delivered"}
        <div>
          <label class="label">
            <span class="label-text">Delivery Date</span>
          </label>
          <input
            type="date"
            bind:value={editedOrder.delivery_date}
            class="input input-bordered w-full"
            max={new Date().toISOString().split('T')[0]}
          />
        </div>
      {/if}
    </div>

    <div class="flex justify-end space-x-2 mt-6">
      <button 
        class="px-4 py-2 bg-red-500 text-white rounded hover:bg-red-600 transition duration-300" 
        on:click={deleteOrder}
      >
        Delete Order
      </button>
      <button 
        class="px-4 py-2 bg-blue-500 text-white rounded hover:bg-blue-600 transition duration-300" 
        on:click={saveChanges}
      >
        Save Changes
      </button>
      <button 
        class="px-4 py-2 bg-gray-300 text-gray-700 rounded hover:bg-gray-400 transition duration-300" 
        on:click={closeModal}
      >
        Cancel
      </button>
    </div>
  </div>
</div>