<script>
    import { onMount } from 'svelte';
    import wilayahs from "../wilayahs.json";

    let dropdownOpen = false;
    
  
    onMount(() => {
      const handleClickOutside = (event) => {
        if (!event.target.closest('.dropdown')) {
          dropdownOpen = false;
        }
      };
      document.addEventListener('click', handleClickOutside);
      return () => {
        document.removeEventListener('click', handleClickOutside);
      };
    });
  </script>
  
  <nav data-sveltekit-reload class="w-full flex gap-3 px-4 py-2 bg-gray-500 text-white">
    <a href="/" class="hover:underline">Home</a>
    <a href="#" class="hover:underline">About</a>
    <div class="dropdown relative">
      <button class="relative z-10 block" on:click={() => dropdownOpen = !dropdownOpen}>
        Wilayah
      </button>
      {#if dropdownOpen}
        <div data-sveltekit-reload class="dropdown-menu absolute left-0 mt-2 w-56 bg-white border border-gray-200 divide-y divide-gray-100 text-black z-20">
          {#each wilayahs.sort() as wilayah (wilayah)}
            <a href="/?wilayah={wilayah}" class="block px-4 py-2 hover:bg-gray-500 hover:text-white" on:click={() => { 
                dropdownOpen = false;
                 }}>{wilayah}</a>
          {/each}
        </div>
      {/if}
    </div>
  </nav>