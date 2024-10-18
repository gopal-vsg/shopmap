<script>
  import { supabase } from "../lib/supabase";
  import { onDestroy } from "svelte";
  import { fade } from 'svelte/transition';

  let name = "";
  let owner = "";
  let phone = "";
  let address = "";
  let lat = "";
  let lng = "";
  let notes = "";
  let audioNote = null;
  let notification = null;
  let mediaRecorder;
  let audioChunks = [];
  
  // Recording state
  let isRecording = false;
  let recordingTime = 0;
  let timer;
  
  // Format seconds to MM:SS
  const formatTime = (seconds) => {
    const mins = Math.floor(seconds / 60).toString().padStart(2, '0');
    const secs = (seconds % 60).toString().padStart(2, '0');
    return `${mins}:${secs}`;
  };

  // Clean up timer on component destruction
  onDestroy(() => {
    if (timer) clearInterval(timer);
  });

  // Show notification
  function showNotification(type, message) {
    notification = { type, message };
    setTimeout(() => {
      notification = null;
    }, 3000);
  }

  function getLocation() {
    if (navigator.geolocation) {
      navigator.geolocation.getCurrentPosition(success, error);
    } else {
      showNotification("error", "Geolocation is not supported by this browser.");
    }
  }

  function success(position) {
    lat = position.coords.latitude.toFixed(4);
    lng = position.coords.longitude.toFixed(4);
    showNotification("success", "Location retrieved successfully!");
  }

  function error() {
    showNotification("error", "Unable to retrieve your location.");
  }

  async function startRecording() {
    try {
      const stream = await navigator.mediaDevices.getUserMedia({ audio: true });
      mediaRecorder = new MediaRecorder(stream);
      audioChunks = [];
      isRecording = true;
      recordingTime = 0;
      
      timer = setInterval(() => {
        recordingTime++;
      }, 1000);

      mediaRecorder.ondataavailable = (event) => {
        audioChunks.push(event.data);
      };

      mediaRecorder.onstop = () => {
        const audioBlob = new Blob(audioChunks, { type: 'audio/mp3' });
        audioNote = audioBlob;
        clearInterval(timer);
        isRecording = false;
      };

      mediaRecorder.start();
    } catch (err) {
      showNotification("error", "Unable to access microphone. Please check permissions.");
    }
  }

  function stopRecording() {
    if (mediaRecorder && isRecording) {
      mediaRecorder.stop();
      showNotification("success", "Recording saved successfully!");
    }
  }

  async function add() {
    if (!name || !owner || !phone || !address || !lat || !lng) {
      showNotification("error", "Please fill in all required fields.");
      return;
    }

    let audioUrl = null;
    if (audioNote) {
      const { data: audioData, error: audioError } = await supabase.storage
        .from("retailer_audio_notes")
        .upload(`retailers/${name}-${Date.now()}.mp3`, audioNote);

      if (audioError) {
        showNotification("error", "Failed to upload audio note: " + audioError.message);
        return;
      }
      audioUrl = audioData.Key;
    }

    const { error } = await supabase
      .from("retailers")
      .insert([{
        name,
        owner,
        phone,
        address,
        lat,
        lng,
        notes,
        audio_note: audioUrl,
      }]);

    if (error) {
      showNotification("error", "Failed to insert values: " + error.message);
    } else {
      showNotification("success", "Retailer added successfully!");
      name = owner = phone = address = lat = lng = notes = "";
      audioNote = null;
      setTimeout(() => location.reload(), 2000);
    }
  }
</script>


<main class="min-h-screen bg-gray-50 py-8 px-4 sm:px-6 lg:px-8">
  <!-- Improved Notification Toast -->
  {#if notification}
    <div
      transition:fade
      class="fixed top-4 right-4 z-50 w-96 p-4 rounded-lg shadow-xl {notification.type === 'success' ? 'bg-green-100 border-l-4 border-green-500' : 'bg-red-100 border-l-4 border-red-500'}"
    >
      <div class="flex items-center justify-between">
        <div class="flex items-center">
          {#if notification.type === 'success'}
            <svg class="h-5 w-5 text-green-500 mr-2" fill="none" viewBox="0 0 24 24" stroke="currentColor">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M5 13l4 4L19 7" />
            </svg>
          {:else}
            <svg class="h-5 w-5 text-red-500 mr-2" fill="none" viewBox="0 0 24 24" stroke="currentColor">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 8v4m0 4h.01M21 12a9 9 0 11-18 0 9 9 0 0118 0z" />
            </svg>
          {/if}
          <p class={notification.type === 'success' ? 'text-green-700 text-sm font-medium' : 'text-red-700 text-sm font-medium'}>
            {notification.message}
          </p>
        </div>
      </div>
    </div>
  {/if}

  <div class="max-w-2xl mx-auto bg-white rounded-xl shadow-lg">
    <div class="px-8 py-6 border-b border-gray-200">
      <h1 class="text-2xl font-bold text-gray-900">Add Retailer Details</h1>
    </div>

    <div class="px-8 py-6">
      <div class="grid grid-cols-1 md:grid-cols-2 gap-6">
        <!-- Left Column -->
        <div class="space-y-5">
          <div>
            <label for="name" class="block text-sm font-medium text-gray-700 mb-1">Name *</label>
            <input
              id="name"
              type="text"
              bind:value={name}
              class="block w-full rounded-md border-gray-300 shadow-sm focus:border-blue-500 focus:ring-blue-500 sm:text-sm"
              placeholder="Enter retailer name"
            />
          </div>

          <div>
            <label for="owner" class="block text-sm font-medium text-gray-700 mb-1">Owner *</label>
            <input
              id="owner"
              type="text"
              bind:value={owner}
              class="block w-full rounded-md border-gray-300 shadow-sm focus:border-blue-500 focus:ring-blue-500 sm:text-sm"
              placeholder="Enter owner name"
            />
          </div>

          <div>
            <label for="phone" class="block text-sm font-medium text-gray-700 mb-1">Phone *</label>
            <input
              id="phone"
              type="tel"
              bind:value={phone}
              class="block w-full rounded-md border-gray-300 shadow-sm focus:border-blue-500 focus:ring-blue-500 sm:text-sm"
              placeholder="Enter phone number"
            />
          </div>
        </div>

        <!-- Right Column -->
        <div class="space-y-5">
          <div>
            <label for="address" class="block text-sm font-medium text-gray-700 mb-1">Address *</label>
            <input
              id="address"
              type="text"
              bind:value={address}
              class="block w-full rounded-md border-gray-300 shadow-sm focus:border-blue-500 focus:ring-blue-500 sm:text-sm"
              placeholder="Enter full address"
            />
          </div>

          <div class="grid grid-cols-2 gap-4">
            <div>
              <label class="block text-sm font-medium text-gray-700 mb-1">Latitude</label>
              <input
                type="text"
                bind:value={lat}
                class="block w-full rounded-md border-gray-300 bg-gray-50 shadow-sm sm:text-sm"
                readonly
              />
            </div>
            <div>
              <label class="block text-sm font-medium text-gray-700 mb-1">Longitude</label>
              <input
                type="text"
                bind:value={lng}
                class="block w-full rounded-md border-gray-300 bg-gray-50 shadow-sm sm:text-sm"
                readonly
              />
            </div>
          </div>

          <button
            on:click={getLocation}
            class="w-full flex items-center justify-center px-4 py-2 border border-transparent rounded-md shadow-sm text-sm font-medium text-white bg-green-600 hover:bg-green-700 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-green-500"
          >
            <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5 mr-2" viewBox="0 0 20 20" fill="currentColor">
              <path fill-rule="evenodd" d="M5.05 4.05a7 7 0 119.9 9.9L10 18.9l-4.95-4.95a7 7 0 010-9.9zM10 11a2 2 0 100-4 2 2 0 000 4z" clip-rule="evenodd" />
            </svg>
            Get Current Location
          </button>
        </div>
      </div>

      <!-- Full Width Elements -->
      <div class="mt-6 space-y-5">
        <div>
          <label for="notes" class="block text-sm font-medium text-gray-700 mb-1">Notes</label>
          <textarea
            id="notes"
            bind:value={notes}
            rows="3"
            class="block w-full rounded-md border-gray-300 shadow-sm focus:border-blue-500 focus:ring-blue-500 sm:text-sm"
            placeholder="Enter any additional notes..."
          ></textarea>
        </div>

        <div class="bg-gray-50 p-4 rounded-lg">
          <label class="block text-sm font-medium text-gray-700 mb-2">Voice Notes</label>
          <div class="flex items-center space-x-4">
            {#if !isRecording}
              <button
                on:click={startRecording}
                class="flex-1 inline-flex items-center justify-center px-4 py-2 border border-transparent rounded-md shadow-sm text-sm font-medium text-white bg-red-600 hover:bg-red-700 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-red-500"
              >
                <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5 mr-2" viewBox="0 0 20 20" fill="currentColor">
                  <path fill-rule="evenodd" d="M7 4a3 3 0 016 0v4a3 3 0 11-6 0V4zm4 10.93A7.001 7.001 0 0017 8a1 1 0 10-2 0A5 5 0 015 8a1 1 0 00-2 0 7.001 7.001 0 006 6.93V17H6a1 1 0 100 2h8a1 1 0 100-2h-3v-2.07z" clip-rule="evenodd" />
                </svg>
                Start Recording
              </button>
            {:else}
              <button
                on:click={stopRecording}
                class="flex-1 inline-flex items-center justify-center px-4 py-2 border border-transparent rounded-md shadow-sm text-sm font-medium text-white bg-red-600 animate-pulse hover:bg-red-700 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-red-500"
              >
                <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5 mr-2" viewBox="0 0 20 20" fill="currentColor">
                  <path fill-rule="evenodd" d="M10 18a8 8 0 100-16 8 8 0 000 16zM8 7a1 1 0 00-1 1v4a1 1 0 001 1h4a1 1 0 001-1V8a1 1 0 00-1-1H8z" clip-rule="evenodd" />
                </svg>
                Stop Recording ({formatTime(recordingTime)})
              </button>
            {/if}
          </div>
          {#if audioNote}
            <p class="mt-2 text-sm text-gray-600">✓ Voice note recorded</p>
          {/if}
        </div>
      </div>

      <div class="mt-6">
        <button
          on:click={add}
          class="w-full flex items-center justify-center px-6 py-3 border border-transparent rounded-md shadow-sm text-base font-medium text-white bg-blue-600 hover:bg-blue-700 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-blue-500"
        >
          <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5 mr-2" viewBox="0 0 20 20" fill="currentColor">
            <path fill-rule="evenodd" d="M10 18a8 8 0 100-16 8 8 0 000 16zm1-11a1 1 0 10-2 0v2H7a1 1 0 100 2h2v2a1 1 0 102 0v-2h2a1 1 0 100-2h-2V7z" clip-rule="evenodd" />
          </svg>
          Add Retailer
        </button>
      </div>
    </div>
  </div>
</main>