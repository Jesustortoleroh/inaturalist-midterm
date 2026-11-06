<script lang="ts">
  // Define the interface for the component's props
  interface Identification {
    name: string;
    scientificName: string;
    image: string;
  }

  // Destructure the item prop with proper TypeScript typing
  let { item }: { item: Identification } = $props();
</script>

<div class="card">
  <div class="img-container">
    <img src={item.image} alt={item.name} />
  </div>
  <h3>{item.name}</h3>
  {#if item.scientificName && item.scientificName !== item.name}
    <p class="scientific-name">{item.scientificName}</p>
  {/if}
</div>

<style>
  .card {
    background: white;
    border: 1px solid #e9ecef;
    border-radius: 8px;
    padding: 1rem;
    text-align: center;
    box-shadow: 0 2px 4px rgba(0,0,0,0.05);
    transition: transform 0.2s;
  }

  .card:hover {
    transform: translateY(-5px);
  }

  .img-container {
    width: 100px;
    height: 100px;
    margin: 0 auto;
    background: #f1f3f5;
    border-radius: 50%;
    overflow: hidden;
    display: flex;
    align-items: center;
    justify-content: center;
    margin-bottom: 1rem;
  }

  .img-container img {
    width: 100%;
    height: 100%;
    object-fit: cover;
  }

  .card h3 {
    font-size: 1rem;
    margin: 0;
    color: #212529;
  }

  .scientific-name {
    font-size: 0.85rem;
    color: #6c757d;
    font-style: italic;
    margin-top: 0.25rem;
  }
</style>