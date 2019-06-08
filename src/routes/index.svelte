<script context="module">
  export function preload(page) {
    return this.fetch("https://meetups-a2909.firebaseio.com/meetups.json")
      .then(res => {
        if (!res.ok) {
          throw new Error("Fetching meetups failed, please try again later!");
        }
        return res.json();
      })
      .then(data => {
        const loadedMeetups = [];
        for (const key in data) {
          loadedMeetups.push({
            ...data[key],
            id: key
          });
        }
        return { fetchedMeetups: loadedMeetups };
        // isLoading = false;
        // meetups.setMeetups(loadedMeetups.reverse());
      })
      .catch(err => {
        error = err;
        isLoading = false;
        console.log(err);
        this.error(500, "Could not fetch meetups!");
      });
  }
</script>

<script>
  import { createEventDispatcher, onMount, onDestroy } from "svelte";
  import { scale } from "svelte/transition";
  import { flip } from "svelte/animate";
  import meetups from "../store/meetups-store.js";
  import MeetupItem from "../components/Meetup/MeetupItem.svelte";
  import MeetupFilter from "../components/Meetup/MeetupFilter.svelte";
  import EditMeetup from "../components/Meetup/EditMeetup.svelte";
  import Button from "../components/UI/Button.svelte";
  import Spinner from "../components/UI/Spinner.svelte";

  export let fetchedMeetups;

  let loadedMeetups = [];
  let editMode;
  let editedId;
  let isLoading;
  let unsubscribe;

  const dispatch = createEventDispatcher();

  let favsOnly = false;

  $: filteredMeetups = favsOnly
    ? loadedMeetups.filter(m => m.isFavorite)
    : loadedMeetups;

  onMount(() => {
    unsubscribe = meetups.subscribe(items => {
      loadedMeetups = items;
    });
    meetups.setMeetups(fetchedMeetups);
  });

  onDestroy(() => {
    if (unsubscribe) {
      unsubscribe();
    }
  });

  const setFilter = event => {
    favsOnly = event.detail === 1;
  };

  const savedMeetup = event => {
    editMode = null;
    editedId = null;
  };

  const cancelEdit = () => {
    editMode = null;
    editedId = null;
  };

  const startEdit = event => {
    editMode = "edit";
    editedId = event.detail;
  };

  const startAdd = () => (editMode = "edit");
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

<svelte:head>
  <title>All Meetups</title>
</svelte:head>

{#if editMode === 'edit'}
  <EditMeetup id={editedId} on:save={savedMeetup} on:cancel={cancelEdit} />
{/if}

{#if isLoading}
  <Spinner />
{:else}
  <section class="meetups-controls">
    <MeetupFilter on:select={setFilter} />
    <Button on:click={startAdd}>New Meetup</Button>
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
{/if}
