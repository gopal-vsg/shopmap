<script>
  import { supabase } from "../lib/supabase";
  import { onMount } from "svelte";
  import L from "leaflet";
  import "leaflet/dist/leaflet.css";

  const customIcon = L.icon({
    iconUrl: "/map-marker.png",
    iconSize: [30, 30],
    iconAnchor: [15, 30],
    popupAnchor: [0, -30],
  });

  let map;
  let markers = {};

  const fetchCoordinates = async () => {
    const { data, error } = await supabase.from("shops").select("name,owner,phone,lat,lng");
    if (error) {
      console.error("Error getting the coordinates", error);
      return [];
    }
    console.log("Coordinates fetched successfully");
    return data;
  };

  const initializeMap = async () => {
    const points = await fetchCoordinates();

    map = L.map("map").setView([13.08884, 80.265372], 13);

    L.tileLayer("https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png", {
      attribution: "© OpenStreetMap contributors",
    }).addTo(map);

    points.forEach((point) => {
      const marker = L.marker([point.lat, point.lng], { icon: customIcon }).addTo(map);
      marker.bindPopup(`<b>${point.name}</b><br>Owner: ${point.owner}<br>Phone: ${point.phone}`);
      markers[`${point.lat},${point.lng}`] = marker;
    });
  };

  export function centerMap(lat, lng) {
    if (map) {
      map.setView([lat, lng], 17);
      const marker = markers[`${lat},${lng}`];
      if (marker) {
        marker.openPopup();
      }
    }
  }

  onMount(() => {
    initializeMap();
  });
</script>

<div class="flex flex-col items-center">
  <p class="  -mt-4 mb-4  text-3xl font-semibold">Shop Map</p>
  <div id="map" class="w-4/6 h-96"></div>
</div>

<style>
  #map {
    height: 550px;
    width: 800px; /* Set a fixed height for the map */
  }
</style>
