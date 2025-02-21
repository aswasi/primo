<script lang="ts">
  import {fade} from 'svelte/transition'
  import Item from './Item.svelte'
  import { PrimaryButton } from '../../../../components/buttons';
  import { TextInput } from '../../../../components/inputs';
  import { pages } from '../../../../stores/data/draft';
  import { pages as actions } from '../../../../stores/actions';
  import activePage, { id as activePageID } from '../../../../stores/app/activePage';
  import { makeValidUrl } from '../../../../utils';
  import { Page } from '../../../../const';

  let currentPath = buildCurrentPath($activePageID);

  function edit_page(pageId, args: { name: string, id: string }) {
    actions.edit(pageId, args)
  }

  async function delete_page(pageId) {
    actions.delete(pageId);
  }

  function add_subpage(pageId) {
    currentPath = [pageId];
    creating_page = false;
    creating_subpage = pageId
  }

  // Page Creation
  let creating_page = false;
  let creating_subpage
  let shouldDuplicatePage = true
  let pageName = '';
  let pageURL = '';

  $: if (!pageLabelEdited) {
    pageURL = makeValidUrl(pageName);
  }
  $: validate_url(pageURL);
  let pageLabelEdited = false;
  $: disablePageCreation = !pageName || !pageURL;
  function validate_url(url) {
    pageURL = url
      .replace(/\s+/g, '-')
      .replace(/[^0-9a-z\-._]/gi, '')
      .toLowerCase();
  }

  async function finish_creating_page() {
    let name = pageName;
    let url = pageURL;
    url = currentPath[0] ? `${currentPath[0]}/${url}` : url; // prepend parent page to id (i.e. about/team)
    console.log({currentPath, url})
    if (shouldDuplicatePage) {
      await actions.duplicate({
        page: $activePage,
        path: currentPath,
        details: { name, id: url }
      })
    } else {
      await actions.add(Page(url, name), currentPath);
    }
    
    creating_page = false;
    creating_subpage = false
    pageName = '';
    pageURL = '';
    shouldDuplicatePage = true;
  }

  function buildCurrentPath(pagePath = '') {
    const [root, child] = pagePath.split('/');
    if (!root || !child) { // on index or top-level page
      return [];
    } else return [root, child];
  }
</script>

<ul class="page-list root">
  {#each $pages as page, i}
    <li>
      <Item 
        {page} 
        active={$activePageID === page.id}
        on:edit={({ detail }) => edit_page(page.id, detail)}
        on:add={({detail:page}) => add_subpage(page.id)}
        on:delete={({detail:page}) => delete_page(page.id)}>
        {#if creating_subpage === page.id}
          <form
            on:submit|preventDefault={() => {
              currentPath = [page.id]
              finish_creating_page()
            }}
            in:fade={{ duration: 100 }}>
            <TextInput
              bind:value={pageName}
              id="page-label"
              label="Page Label"
              placeholder="About Us" />
            <TextInput
              bind:value={pageURL}
              id="page-url"
              label="Page URL"
              prefix="/"
              on:input={() => (pageLabelEdited = true)}
              placeholder="about-us" />
            <div id="duplicate">
              <label>
                <input bind:checked={shouldDuplicatePage} type="checkbox">
                <span>Duplicate active page</span>
              </label>
            </div>
            <PrimaryButton
              disabled={disablePageCreation}
              id="create-page"
              type="submit">
              Create
            </PrimaryButton>
          </form>
        {/if}
      </Item>
    </li>
  {/each}
  {#if creating_page}
    <li>
      <form
        on:submit|preventDefault={() => {
          currentPath = []
          finish_creating_page()
        }}
        in:fade={{ duration: 100 }}>
        <TextInput
          bind:value={pageName}
          id="page-label"
          label="Page Label"
          placeholder="About Us" />
        <TextInput
          bind:value={pageURL}
          id="page-url"
          label="Page URL"
          prefix="/"
          on:input={() => (pageLabelEdited = true)}
          placeholder="about-us" />
        <div id="duplicate">
          <label>
            <input bind:checked={shouldDuplicatePage} type="checkbox">
            <span>Duplicate active page</span>
          </label>
        </div>
        <PrimaryButton
          disabled={disablePageCreation}
          id="create-page"
          type="submit">
          Create
        </PrimaryButton>
      </form>
    </li>
  {/if}
</ul>
{#if !creating_page}
  <PrimaryButton on:click={() => (creating_page = true)} label="Create Page" icon="akar-icons:plus" />
{/if}

<style lang="postcss">
  ul.page-list {
    display: grid;
    color: var(--primo-color-white);
    background: #252627;
    border-radius: var(--primo-border-radius);
    margin-bottom: 1rem;

    &.root > li:not(:first-child) {
      border-top: 1px solid #3E4041;
    }
  }
  form {
    padding: 0.25rem;
    display: grid;
    gap: 0.5rem;
    max-width: 400px;
    padding: 0.825rem 1.125rem;
    margin-bottom: 0.5rem;
  }
</style>