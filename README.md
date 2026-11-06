# iNaturalist Search App - Midterm Exercise

A dynamic web application built with **Svelte 5**, **TypeScript**, and **Vite** that consumes the [iNaturalist API](https://api.inaturalist.org/v1/). 

## Features

- **Place Search:** Users can search for locations using the iNaturalist Places autocomplete API.
- **Species Observations:** Fetches real-time observations based on the selected place.
- **Beautiful UI:** Displays species in a responsive grid with their common name, scientific name, and a photo.
- **Modular Components:** Uses reusable Svelte 5 components (`IdentificationCard.svelte`) with strictly typed props.
- **Robust Error Handling:** Displays user-friendly error messages if the API request fails, and handles loading states gracefully.

## Stretch Goals Completed

- **De-duplication:** Uses a JavaScript `Map` to filter out duplicate taxons from the observations results, ensuring each species only appears once.
- **Date Filtering:** Includes "From" and "To" date inputs that pass the `d1` and `d2` parameters to the iNaturalist API to filter observations by date.

## Tech Stack

- **Framework:** Svelte 5 (using Runes: `$state`, `$props`)
- **Language:** TypeScript (with strict interfaces)
- **Build Tool:** Vite
- **API:** iNaturalist REST API

## How to Run Locally

1. Clone the repository:
   ```bash
   git clone https://github.com/Jesustortoleroh/inaturalist-midterm.git