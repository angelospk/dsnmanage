<script>
  import { onMount } from 'svelte';
  
  export let ideasUrl;
  export let currentSessionDate;
  
  let ideas = [];
  let loading = true;
  let error = null;
  
  onMount(async () => {
    try {
      const response = await fetch(ideasUrl);
      if (!response.ok) {
        throw new Error('Failed to fetch ideas');
      }
      ideas = await response.json();
      console.log('Loaded ideas:', ideas); // Debug log
    } catch (e) {
      error = 'Σφάλμα κατά τη φόρτωση των ιδεών. Παρακαλώ δοκιμάστε ξανά αργότερα.';
      console.error('Error fetching ideas:', e);
    } finally {
      loading = false;
    }
  });
  
  // Function to check if an idea is within the 3-day window before the session
  function isWithinSubmissionWindow(timestamp) {
    if (!currentSessionDate) return false;
    const submissionDate = new Date(timestamp);
    //TO DO: load and use the actual session date
    //add sessionDate the date 04/05/2025
    const sessionDate = new Date('2025-04-05T21:00:00.000Z');
    const threeDaysBefore = new Date(sessionDate);
    threeDaysBefore.setDate(sessionDate.getDate() - 3);
    
    return submissionDate <= threeDaysBefore && submissionDate <= sessionDate;
  }
  
  // Filter ideas to only show those within the submission window
  $: filteredIdeas = ideas.filter(idea => 
    isWithinSubmissionWindow(idea["Χρονική σήμανση"])
  );

  $: console.log('Filtered ideas:', filteredIdeas); // Debug log
</script>

<div class="w-full">
  {#if loading}
    <div class="space-y-4">
      {#each Array(3) as _}
        <div class="bg-white p-6 rounded-lg shadow-sm border border-gray-200 animate-pulse">
          <div class="h-4 bg-gray-200 rounded w-3/4 mb-4"></div>
          <div class="h-3 bg-gray-200 rounded w-1/2"></div>
        </div>
      {/each}
    </div>
  {:else if error}
    <div class="bg-red-50 border-l-4 border-red-400 p-4 rounded">
      <div class="flex">
        <div class="flex-shrink-0">
          <svg class="h-4 w-4 text-red-400" viewBox="0 0 20 20" fill="currentColor">
            <path fill-rule="evenodd" d="M10 18a8 8 0 100-16 8 8 0 000 16zM8.707 7.293a1 1 0 00-1.414 1.414L8.586 10l-1.293 1.293a1 1 0 101.414 1.414L10 11.414l1.293 1.293a1 1 0 001.414-1.414L11.414 10l1.293-1.293a1 1 0 00-1.414-1.414L10 8.586 8.707 7.293z" clip-rule="evenodd" />
          </svg>
        </div>
        <div class="ml-2">
          <p class="text-sm text-red-700">{error}</p>
        </div>
      </div>
    </div>
  {:else if filteredIdeas.length === 0}
    <div class="bg-yellow-50 border-l-4 border-yellow-400 p-4 rounded">
      <div class="flex">
        <div class="flex-shrink-0">
          <svg class="h-4 w-4 text-yellow-400" viewBox="0 0 20 20" fill="currentColor">
            <path fill-rule="evenodd" d="M8.257 3.099c.765-1.36 2.722-1.36 3.486 0l5.58 9.92c.75 1.334-.213 2.98-1.742 2.98H4.42c-1.53 0-2.493-1.646-1.743-2.98l5.58-9.92zM11 13a1 1 0 11-2 0 1 1 0 012 0zm-1-8a1 1 0 00-1 1v3a1 1 0 002 0V6a1 1 0 00-1-1z" clip-rule="evenodd" />
          </svg>
        </div>
        <div class="ml-2">
          <p class="text-sm text-yellow-700">
            Δεν βρέθηκαν ιδέες για την τρέχουσα συνεδρίαση.
          </p>
        </div>
      </div>
    </div>
  {:else}
    <div class="space-y-4">
      {#each filteredIdeas as idea}
        <div class="bg-white rounded-lg shadow-sm border border-gray-200 overflow-hidden">
          <div class="p-6">
            <p class="text-lg font-medium text-gray-900 mb-3">
              {idea["Γράψε αναλυτικά την ιδέα σου:"]}
            </p>
            <p class="text-sm text-gray-600">
              Από: <span class="font-medium">{idea["Όνομα:"]}</span>
            </p>
          </div>
          
          <details class="border-t border-gray-200">
            <summary class="px-6 py-4 cursor-pointer hover:bg-gray-50 flex items-center justify-between">
              <span class="text-sm text-gray-600">Περισσότερα:</span>
              <svg class="h-4 w-4 text-gray-400" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 9l-7 7-7-7" />
              </svg>
            </summary>
            <div class="px-6 py-4 bg-gray-50">
              <dl class="grid grid-cols-1 gap-4 sm:grid-cols-2">
                <div>
                  <dt class="text-sm font-medium text-gray-500">Email</dt>
                  <dd class="mt-1 text-sm text-gray-900">{idea["E-mail:"]}</dd>
                </div>
                <div>
                  <dt class="text-sm font-medium text-gray-500">Τηλέφωνο</dt>
                  <dd class="mt-1 text-sm text-gray-900">{idea["Τηλέφωνο:"]}</dd>
                </div>
                <div class="col-span-full">
                  <dt class="text-sm font-medium text-gray-500">Χρονική σήμανση</dt>
                  <dd class="mt-1 text-sm text-gray-900">
                    {new Date(idea["Χρονική σήμανση"]).toLocaleString('el-GR')}
                  </dd>
                </div>
              </dl>
            </div>
          </details>
        </div>
      {/each}
    </div>
  {/if}
</div>

<style>
  details {
    transition: all 0.2s ease;
  }
  
  details[open] summary svg {
    transform: rotate(180deg);
  }
  
  summary {
    list-style: none;
  }
  
  summary::-webkit-details-marker {
    display: none;
  }
</style> 