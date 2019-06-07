<script>
  import meetups from "./Meetups/meetups-store.js";

  import Header from "./UI/Header.svelte";
  import TextInput from "./UI/TextInput.svelte";
  import Button from "./UI/Button.svelte";
  import Spinner from "./UI/Spinner.svelte";
  import Error from "./UI/Error.svelte";

  import MeetupGrid from "./Meetups/MeetupGrid.svelte";
  import EditMeetup from "./Meetups/EditMeetup.svelte";
  import MeetupDetail from "./Meetups/MeetupDetail.svelte";

  let editMode;
  let editeId;
  let page = "overview";
  let pageData = {};
  let isLoading = true;
  let error;

  fetch("https://meetups-a2909.firebaseio.com/meetups.json")
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
      isLoading = false;
      meetups.setMeetups(loadedMeetups.reverse());
    })
    .catch(err => {
      error = err;
      isLoading = false;
      console.log(err);
    });

  const savedMeetup = event => {
    editMode = null;
    editeId = null;
  };

  const cancelEdit = () => {
    editMode = null;
    editeId = null;
  };

  const showdetails = event => {
    page = "details";
    pageData.id = event.detail;
  };

  const closeDetails = () => {
    page = "overview";
    pageData = {};
  };

  const startEdit = event => {
    editMode = "edit";
    editeId = event.detail;
  };

  const clearError = () => {
    error = null;
  };
</script>

<style>
  main {
    margin-top: 5rem;
  }
</style>

{#if error}
  <Error message={error.message} on:cancel={clearError} />
{/if}

<Header />

<main>
  {#if page === 'overview'}
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
  {:else}
    <MeetupDetail id={pageData.id} on:close={closeDetails} />
  {/if}
</main>
