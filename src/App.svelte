<script>
  import meetups from "./Meetups/meetups-store.js";

  import Header from "./UI/Header.svelte";
  import TextInput from "./UI/TextInput.svelte";
  import Button from "./UI/Button.svelte";
  import MeetupGrid from "./Meetups/MeetupGrid.svelte";
  import EditMeetup from "./Meetups/EditMeetup.svelte";
  import MeetupDetail from "./Meetups/MeetupDetail.svelte";

  let editMode;
  let editeId;
  let page = "overview";
  let pageData = {};

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
</script>

<style>
  main {
    margin-top: 5rem;
  }
</style>

<Header />

<main>
  {#if page === 'overview'}
    {#if editMode === 'edit'}
      <EditMeetup id={editeId} on:save={savedMeetup} on:cancel={cancelEdit} />
    {/if}
    <MeetupGrid
      meetups={$meetups}
      on:showdetails={showdetails}
      on:edit={startEdit}
      on:add={() => {
        editMode = 'edit';
      }} />
  {:else}
    <MeetupDetail id={pageData.id} on:close={closeDetails} />
  {/if}
</main>
