<script lang="ts">
  import { onMount } from 'svelte'

  type Row = Record<string, string>

  let rows: Row[] = []
  let query = ''
  let loading = true
  let errorMessage = ''

  onMount(async () => {
    try {
      const response = await fetch(`${import.meta.env.BASE_URL}hotwheels.json`)
      if (!response.ok) {
        throw new Error(`Failed to load data (${response.status})`)
      }
      rows = await response.json()
    } catch (error) {
      errorMessage = error instanceof Error ? error.message : 'Failed to load data.'
    } finally {
      loading = false
    }
  })

  $: normalizedQuery = query.trim().toLowerCase()
  $: visibleRows = rows
    .filter((row) => {
      if (!normalizedQuery) return true
      const name = (row.Name ?? '').toString().toLowerCase()
      const toyNumber = (row.Number ?? '').toString().toLowerCase()
      return name.includes(normalizedQuery) || toyNumber.startsWith(normalizedQuery)
    })
    .slice(0, 20)
</script>

<main class="page">
  <header class="toolbar">
    <label class="search-label" for="search">Search</label>
    <input
      id="search"
      type="search"
      placeholder="Name or Toy # (starts with)"
      bind:value={query}
      autocomplete="off"
      spellcheck="false"
    />
  </header>

  <section class="results">
    {#if loading}
      <div class="state">Loading rows...</div>
    {:else if errorMessage}
      <div class="state error">{errorMessage}</div>
    {:else}
      <div class="table-shell">
        <table>
          <thead>
            <tr>
              <th>Number</th>
              <th>Name</th>
              <th>Year</th>
              <th>Color</th>
            </tr>
          </thead>
          <tbody>
            {#each visibleRows as row}
              <tr>
                <td data-label="Name">{row.Name}</td>
                <td class="mono" data-label="Number">{row.Number}</td>
                <td data-label="Year">{row.Year}</td>
                <td data-label="Color">{row.Color}</td>
              </tr>
            {/each}
          </tbody>
        </table>
      </div>
    {/if}
  </section>
</main>
