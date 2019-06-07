<script>
  import { createEventDispatcher } from "svelte";

  import meetups from "./meetups-store.js";
  import { isEmpty, isValidEmail } from "../helpers/validation.js";

  import TextInput from "../UI/TextInput.svelte";
  import Button from "../UI/Button.svelte";
  import Modal from "../UI/Modal.svelte";

  export let id = null;

  let title = "";
  let subtitle = "";
  let address = "";
  let email = "";
  let description = "";
  let imageUrl = "";

  if (id) {
    const unsubscribe = meetups.subscribe(items => {
      const selectedMeetup = items.find(i => i.id === id);
      title = selectedMeetup.title;
      subtitle = selectedMeetup.subtitle;
      address = selectedMeetup.address;
      email = selectedMeetup.contactEmail;
      description = selectedMeetup.description;
      imageUrl = selectedMeetup.imageUrl;
    });

    unsubscribe();
  }

  const dispatch = createEventDispatcher();

  $: titleValid = !isEmpty(title);
  $: subtitleValid = !isEmpty(subtitle);
  $: addressValid = !isEmpty(address);
  $: emailValid = isValidEmail(email);
  $: descriptionValid = !isEmpty(description);
  $: imageUrlValid = !isEmpty(imageUrl);
  $: formIsValid =
    titleValid &&
    subtitleValid &&
    addressValid &&
    emailValid &&
    descriptionValid &&
    imageUrlValid;

  const submitForm = () => {
    const meetupData = {
      title,
      subtitle,
      description,
      imageUrl,
      contactEmail: email,
      address
    };

    // meetups.push(newMeetup); // DOES NOT WORK!
    if (id) {
      fetch(`https://meetups-a2909.firebaseio.com/meetups/${id}.json`, {
        method: "PATCH",
        body: JSON.stringify(meetupData),
        headers: { "Content-Type": "application/json" }
      })
        .then(res => {
          if (!res.ok) {
            throw new Error("Failed");
          }
          meetups.updateMeetup(id, meetupData);
        })
        .catch(err => {
          color.log(err);
        });
    } else {
      fetch("https://meetups-a2909.firebaseio.com/meetups.json", {
        method: "POST",
        body: JSON.stringify({ ...meetupData, isFavorite: false }),
        headers: { "Content-Type": "application/json" }
      })
        .then(res => {
          if (!res.ok) {
            throw new Error("Failed");
          }
          res.json();
        })
        .then(data => {
          meetups.addMeetup({
            ...meetupData,
            isFavorite: false,
            id: data.name
          });
        })
        .catch(err => {
          color.log(err);
        });
    }
    dispatch("save");
  };

  const cancel = () => {
    dispatch("cancel");
  };

  const deleteMeetup = () => {
    meetups.removeMeetup(id);
    dispatch("save");
  };
</script>

<style>
  form {
    max-width: 100%;
  }
</style>

<Modal title="Edit Meetup Data" on:cancel>
  <form on:submit|preventDefault={submitForm}>
    <TextInput
      id="title"
      label="Title"
      valid={titleValid}
      validityMessage="Please enter a valid title."
      value={title}
      on:input={event => (title = event.target.value)} />
    <TextInput
      id="subtitle"
      label="Subtitle"
      valid={subtitleValid}
      validityMessage="Please enter a valid subtitle."
      value={subtitle}
      on:input={event => (subtitle = event.target.value)} />
    <TextInput
      id="address"
      label="Address"
      valid={addressValid}
      validityMessage="Please enter a valid address."
      value={address}
      on:input={event => (address = event.target.value)} />
    <TextInput
      id="imageUrl"
      label="Image URL"
      valid={imageUrlValid}
      validityMessage="Please enter a valid image url."
      value={imageUrl}
      on:input={event => (imageUrl = event.target.value)} />
    <TextInput
      id="email"
      label="E-Mail"
      valid={emailValid}
      validityMessage="Please enter a valid e-mail."
      type="email"
      value={email}
      on:input={event => (email = event.target.value)} />
    <TextInput
      id="description"
      label="Description"
      valid={descriptionValid}
      validityMessage="Please enter a valid description."
      controlType="textarea"
      bind:value={description} />
  </form>
  <div slot="footer">
    <Button type="button" mode="outline" on:click={cancel}>Cancel</Button>
    <Button type="button" on:click={submitForm} disabled={!formIsValid}>
      Save
    </Button>
    {#if id}
      <Button type="button" on:click={deleteMeetup}>Delete</Button>
    {/if}
  </div>
</Modal>
