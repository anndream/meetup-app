<script>
  import { createEventDispatcher } from "svelte";
  import { scale } from "svelte/transition";
  import { flip } from "svelte/animate";
  import MeetupItem from "./MeetupItem.svelte";
  import MeetupFilter from "./MeetupFilter.svelte";
  import Button from "../UI/Button.svelte";
  export let meetups;
  const dispatch = createEventDispatcher();
  let favsOnly = false;
  $: filteredMeetups = favsOnly ? meetups.filter(m => m.isFavorite) : meetups;
  const setFilter = event => {
    favsOnly = event.detail === 1;
  };
</script>

<style>
  .meetups {
    width: 100%;
    display: grid;
    grid-template-columns: 1fr;
    grid-gap: 1rem;
  }
  .meetups-controls {
    margin: 1rem;
    display: flex;
    justify-content: space-between;
  }
  @media (min-width: 768px) {
    .meetups {
      grid-template-columns: repeat(2, 1fr);
    }
  }
  .no-meetups {
    margin: 1rem;
  }
</style>

{#if editMode === 'edit'}
  <EditMeetup id={editeId} on:save={savedMeetup} on:cancel={cancelEdit} />
{/if}
{#if isLoading}
  <Spinner />
{:else}
  <MeetupGrid
    meetups={$meetups}
    on:showdetails={showdetails}
    on:edit={startEdit}
    on:add={() => {
      editMode = 'edit';
    }} />
{/if}

<section class="meetups-controls">
  <MeetupFilter on:select={setFilter} />
  <Button on:click={() => dispatch('add')}>New Meetup</Button>
</section>
{#if filteredMeetups.length === 0}
  <p class="no-meetups">No meetups found, you can start adding some.</p>
{/if}
<section class="meetups">
  {#each filteredMeetups as meetup (meetup.id)}
    <div transition:scale animate:flip={{ duration: 300 }}>
      <MeetupItem
        id={meetup.id}
        title={meetup.title}
        subtitle={meetup.subtitle}
        imageUrl={meetup.imageUrl}
        description={meetup.description}
        email={meetup.contactEmail}
        address={meetup.address}
        isFav={meetup.isFavorite}
        on:showdetails
        on:edit />
    </div>
  {/each}
</section>
