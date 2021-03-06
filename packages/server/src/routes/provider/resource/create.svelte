<script context="module">
  export async function preload({ params, query }) {
    const subcategoryResponse = await this.fetch(`/api/subcategories`);
    const { subcategories } = await subcategoryResponse.json();
    if (subcategoryResponse.status !== 200) {
      this.error(subcategoryResponse.status, subcategoryData.message);
    } else {
      return { subcategories };
    }
  }
</script>

<script>
  import { addFlashMessage } from "./../../../utility/flashMessage.ts";
  import { goto, stores } from "@sapper/app";
  import { ResourceSchedule } from "@upswyng/common";
  import ResourceEditor from "./../../../components/ResourceEditor.svelte";

  const { session } = stores();

  export let subcategories; // TSubcategory[], all subcategories in the app

  let isSaving = false;
  let saveError = null; // Error | null

  let resource /* TResource */ = {
    address: { address1: "", address2: "", city: "", state: "", zip: "" },
    closeSchedule: [],
    description: "",
    latitude: 40.01,
    longitude: -105.27,
    name: "",
    phone: "",
    schedule: new ResourceSchedule(),
    services: [],
    website: "",
  };

  function handleSaveClick(resource) {
    saveError = null;
    isSaving = true;

    const options = {
      method: "POST",
      body: JSON.stringify({ draftResource: resource }),
      headers: {
        "Content-Type": "application/json",
      },
    };

    fetch("/api/resource", options)
      .then(async res => {
        if (res.status >= 400) {
          const { message } = await res.json();
          throw new Error(
            message || "There was an error creating the draft resource."
          );
        }
        return await res.json();
      })
      .then(res => {
        if (res.draftResource) {
          addFlashMessage(
            session,
            "success",
            res.draftResource.name
              ? `A draft of ${res.draftResource.name} was created`
              : "A new draft was created"
          );
          goto("/provider/resource");
        } else {
          console.error(res);
          saveError = new Error(
            "There was an error creating the draft service provider."
          );
        }
      })
      .catch(e => (saveError = e))
      .finally(() => (isSaving = false));
  }
</script>

<svelte:head>
  <title>Upswyng: Create a New Service Provider</title>
</svelte:head>

<section class="section">
  <div class="container">
    <h1 class="title">Create A Service Provider</h1>
    <ResourceEditor
      {resource}
      {subcategories}
      errorText={saveError ? saveError.message : ''}
      on:dispatchSaveResource={e => handleSaveClick(e.detail)} />
  </div>
</section>
