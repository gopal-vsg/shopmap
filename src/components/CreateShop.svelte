<script>
  import { supabase } from "../lib/supabase";

  let name = "";
  let owner = "";
  let phone = "";
  let address = "";
  let lat = "";
  let lng = "";
  let message = "";
  let messageType = ""; 

  async function add() {
    if (name && owner && phone && address && lat && lng) {
      const { data, error } = await supabase
        .from("retailers")  // Changed from 'shops' to 'retailers'
        .insert([
          {
            name: name,
            owner: owner,
            phone: phone,
            address: address,
            lat: lat,
            lng: lng,
          },
        ]);

      if (error) {
        messageType = "error";
        message = "Failed to insert values: " + error.message;
      } else {
        messageType = "success";
        message = "Values added to the table successfully.";
        name = "";
        owner = "";
        phone = "";
        address = "";
        lat = "";
        lng = "";
        location.reload();
      }
    } else {
      messageType = "error";
      message = "Please fill in all fields.";
    }
    // @ts-ignore
    my_modal_2.showModal();
  }
</script>

<main class="flex flex-col items-center justify-center bg-gray-100 p-4">
  <h1 class="text-2xl font-bold mb-4">Add Retailer Details</h1>  <!-- Changed 'shop' to 'retailer' -->

  <div class="space-y-4">
    <input type="text" placeholder="Name" bind:value={name} class="w-full p-2 border border-gray-300 rounded" />
    <input type="text" placeholder="Owner" bind:value={owner} class="w-full p-2 border border-gray-300 rounded" />
    <input type="text" placeholder="Phone" bind:value={phone} class="w-full p-2 border border-gray-300 rounded" />
    <input type="text" placeholder="Address" bind:value={address} class="w-full p-2 border border-gray-300 rounded" />
    <input type="text" placeholder="Latitude" bind:value={lat} class="w-full p-2 border border-gray-300 rounded" />
    <input type="text" placeholder="Longitude" bind:value={lng} class="w-full p-2 border border-gray-300 rounded" />
  </div>
  <button on:click={add} class="w-full mt-10 bg-blue-500 text-white p-2 rounded hover:bg-blue-600">Add Retailer</button>  <!-- Changed 'shop' to 'retailer' -->
  

  <dialog id="my_modal_2" class="modal">
      <div class="modal-box">
        <h3 class="text-lg font-bold">{messageType}</h3>
        <p class="py-4">{message}</p>
      </div>
      <form method="dialog" class="modal-backdrop">
        <button>close</button>
      </form>
    </dialog>
</main>
