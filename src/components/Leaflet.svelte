<script>
    import { supabase } from "../lib/supabase";
    import { onMount } from "svelte";
    import L from "leaflet";
    import "leaflet/dist/leaflet.css";
  
    let map;
  
    const fetchCoordinates = async () => {
      const { data, error } = await supabase.from("shops").select("lat, lng");
      if (error) {
        console.error("Error getting the coordinates", error);
        return [];
      } else {
        console.log("Coordinates fetched successfully");
        return data;
      }
    };
  
    const initializeMap = async () => {
      const points = await fetchCoordinates();
  
      map = L.map("map").setView([13.08884, 80.265372], 13);
  
      L.tileLayer("https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png", {
        attribution: "© OpenStreetMap contributors",
      }).addTo(map);
  
      points.forEach((point) => {
        L.marker([point.lat, point.lng]).addTo(map);
      });
    };
  
    onMount(() => {
      initializeMap();
    });
  </script>
  
  <div class="flex flex-col items-center">
    <p class="m-4 text-3xl font-semibold">Shop Map</p>
    <div id="map" class="size-4/6"></div>
  </div>
  
  <style>
    #map {
      height: 400px;
    }
  </style>
  