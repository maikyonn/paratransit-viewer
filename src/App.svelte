<script>
  import { onMount } from "svelte";
  import L from "leaflet";

  let map;
  let geoJsonLayer;
  let services = [];
  let selectedService = null;

  async function fetchServices() {
    const response = await fetch("/service_zones/services.json");
    const data = await response.json();
    services = Object.keys(data);
  }

  async function updateMap(service) {
    if (!service) return;

    const response = await fetch(
      `/service_zones/zones2/${service.replace(/ /g, "_")}.geojson`
    );
    const geoJsonData = await response.json();
``
    if (geoJsonLayer) {
      geoJsonLayer.remove();
    }

    geoJsonLayer = L.geoJSON(geoJsonData, {
      style: {
        color: "#007BFF",
        weight: 2,
        fillOpacity: 0.2,
      },
    }).addTo(map);

    map.fitBounds(geoJsonLayer.getBounds(), {
      padding: [50, 50],
      maxZoom: 16,
      duration: 0.5,
      animate: true,
    });
  }

  onMount(() => {
    map = L.map("map").setView([37.8, -122.3], 9);

    L.tileLayer("https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png", {
      maxZoom: 19,
      attribution:
        '© <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors',
    }).addTo(map);

    fetchServices();
  });
</script>

<div class="container">
  <div class="controls">
    <label for="services">Select a Service:</label>
    <select
      id="services"
      bind:value={selectedService}
      on:change={() => updateMap(selectedService)}
    >
      <option value="" disabled selected>Select a service</option>
      {#each services as service}
        <option value={service}>{service}</option>
      {/each}
    </select>
  </div>
  <div id="map"></div>
</div>

<style>
  .container {
    position: relative;
    width: 100%;
    height: 100vh;
  }

  #map {
    width: 100%;
    height: 100%;
  }

  .controls {
    position: absolute;
    top: 20px;
    left: 20px;
    z-index: 1000;
    background-color: white;
    padding: 12px;
    border-radius: 8px;
    box-shadow: 0 2px 6px rgba(0, 0, 0, 0.3);
  }

  .controls label {
    display: block;
    margin-bottom: 8px;
    font-weight: bold;
    color: #333;
  }

  .controls select {
    width: 200px;
    padding: 8px;
    border: 1px solid #ccc;
    border-radius: 4px;
    background-color: white;
    font-size: 14px;
  }

  .controls select:focus {
    outline: none;
    border-color: #007bff;
    box-shadow: 0 0 0 2px rgba(0, 123, 255, 0.25);
  }
</style>
