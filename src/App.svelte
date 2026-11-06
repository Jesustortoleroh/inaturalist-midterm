<script lang="ts">
  import IdentificationCard from './lib/IdentificationCard.svelte';

  // Define interfaces for the data structures
  interface Place {
    id: number;
    display_name: string;
  }

  interface Identification {
    id: number;
    name: string;
    scientificName: string;
    image: string;
  }

  let searchQuery = $state("");
  let places = $state<Place[]>([]);
  let selectedPlaceId = $state("");
  let identifications = $state<Identification[]>([]);
  let startDate = $state("");
  let endDate = $state("");
  let isLoading = $state(false);
  let errorMessage = $state("");

  // Search for places based on user input
  async function searchPlaces() {
    if (!searchQuery) return;
    errorMessage = ""; // Clear previous errors
    
    try {
      const res = await fetch(`https://api.inaturalist.org/v1/places/autocomplete?q=${searchQuery}`);
      
      // Expert tip: fetch does not reject on HTTP errors, so we must check 'res.ok'
      if (!res.ok) throw new Error(`HTTP error! status: ${res.status}`);
      
      const data = await res.json();
      places = data.results; 
    } catch (error) {
      console.error("Error fetching places:", error);
      errorMessage = "Failed to search for places. Please try again.";
    }
  }

  // Search for observations using the correct endpoint
  async function searchIdentifications() {
    if (!selectedPlaceId) return;
    isLoading = true;
    identifications = []; 
    errorMessage = "";
    
    try {
      let url = `https://api.inaturalist.org/v1/observations?place_id=${selectedPlaceId}&per_page=30&order_by=observed_on`;
      
      if (startDate) url += `&d1=${startDate}`;
      if (endDate) url += `&d2=${endDate}`;

      const res = await fetch(url);
      
      // Check response status before parsing
      if (!res.ok) throw new Error(`HTTP error! status: ${res.status}`);

      const data = await res.json();
      const uniqueTaxons = new Map();
      
      data.results.forEach((item: any) => {
        if (item.taxon && item.taxon.id) {
          
          let imageUrl = 'https://via.placeholder.com/100?text=No+Image';
          
          if (item.taxon.default_photo && item.taxon.default_photo.square_url) {
            imageUrl = item.taxon.default_photo.square_url;
          } 
          else if (item.photos && item.photos.length > 0 && item.photos[0].url) {
            imageUrl = item.photos[0].url.replace('thumb', 'square');
          }

          if (!uniqueTaxons.has(item.taxon.id)) {
            uniqueTaxons.set(item.taxon.id, {
              id: item.taxon.id,
              name: item.taxon.preferred_common_name || item.taxon.name || "Unknown Species",
              scientificName: item.taxon.name,
              image: imageUrl
            });
          }
        }
      });
      
      identifications = Array.from(uniqueTaxons.values());
    } catch (error) {
      console.error("Error fetching observations:", error);
      errorMessage = "Failed to load species. Please check your connection and try again.";
    } finally {
      isLoading = false;
    }
  }
</script>

<main class="container">
  <h1>iNaturalist Search</h1>
  <p>Search for a place and view the species observed there.</p>

  <!-- Display error message if an API call fails -->
  {#if errorMessage}
    <div class="error-banner">
      {errorMessage}
    </div>
  {/if}

  <div class="search-section">
    <div class="form-group">
      <input 
        bind:value={searchQuery} 
        onkeydown={(e) => e.key === 'Enter' && searchPlaces()} 
        placeholder="Search for a place (e.g., Idaho, Yellowstone)..." 
      />
      <button onclick={searchPlaces}>Search</button>
    </div>

    {#if places.length > 0}
      <div class="filters-section">
        <div class="form-group">
          <label for="place-select">Select a place:</label>
          <select id="place-select" bind:value={selectedPlaceId}>
            <option value="">-- Choose a place --</option>
            {#each places as place}
              <option value={place.id}>{place.display_name}</option>
            {/each}
          </select>
        </div>
        
        <div class="date-filters">
          <div class="form-group">
            <!-- Added explicit whitespace (&nbsp;) to prevent Svelte 5 from collapsing the space before the input -->
            <label>From:&nbsp;<input type="date" bind:value={startDate} /></label>
            <label>To:&nbsp;<input type="date" bind:value={endDate} /></label>
          </div>
        </div>

        <button onclick={searchIdentifications} disabled={!selectedPlaceId || isLoading}>
          {isLoading ? 'Loading...' : 'Get Species'}
        </button>
      </div>
    {/if}
  </div>

  <!-- Display Results using the new component -->
  <div class="results-grid">
    {#each identifications as item}
      <IdentificationCard {item} />
    {/each}
  </div>
</main>

<style>
  .container {
    max-width: 900px;
    margin: 0 auto;
    padding: 2rem;
    font-family: system-ui, -apple-system, sans-serif;
    color: #f8f9fa; 
  }

  h1 {
    color: #2a9d8f;
    text-align: center;
  }

  /* Styles for the new error banner */
  .error-banner {
    background-color: #e63946;
    color: white;
    padding: 1rem;
    border-radius: 8px;
    margin-bottom: 1rem;
    text-align: center;
    font-weight: bold;
  }
  
  .search-section {
    background: #2b2b2b; 
    padding: 1.5rem;
    border-radius: 8px;
    margin-bottom: 2rem;
    border: 1px solid #444;
  }

  .form-group {
    display: flex;
    gap: 10px;
    margin-bottom: 1rem;
    align-items: center;
  }

  input, input[type="date"], select {
    padding: 0.5rem;
    border: 1px solid #ccc;
    border-radius: 4px;
    flex: 1;
    font-size: 1rem;
  }

  button {
    padding: 0.5rem 1rem;
    background-color: #2a9d8f;
    color: white;
    border: none;
    border-radius: 4px;
    cursor: pointer;
    font-size: 1rem;
    font-weight: bold;
    transition: background 0.2s;
  }

  button:hover:not(:disabled) {
    background-color: #21867a;
  }

  button:disabled {
    background-color: #ccc;
    cursor: not-allowed;
  }

  .date-filters {
    margin-top: 1rem;
    border-top: 1px dashed #ccc;
    padding-top: 1rem;
  }

  .results-grid {
    display: grid;
    grid-template-columns: repeat(auto-fill, minmax(160px, 1fr));
    gap: 1.5rem;
  }
</style>