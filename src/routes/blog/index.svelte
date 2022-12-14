<script context="module">
  export const prerender = true; // turned off so it refreshes quickly
  export async function load({ fetch }) {
    const res = await fetch(`/api/listContent.json`);
    // alternate strategy https://www.davidwparker.com/posts/how-to-make-an-rss-feed-in-sveltekit
    // Object.entries(import.meta.glob('./*.md')).map(async ([path, page]) => {
    if (res.status > 400) {
      return {
        status: res.status,
        error: await res.text()
      };
    }

    /** @type {import("$lib/types").ContentItem[]} */
    const items = await res.json();
    return {
      props: { items },
      maxage: 60 // 1 minute
    };
  }
</script>

<script>
  import IndexCard from "$lib/components/IndexCard.svelte";
  import dayjs from "dayjs";
  import { SITE_TITLE } from "$lib/siteConfig";


  export let list;
  export let items;

  let inputEl;

  function focusSearch(e) {
    if (e.key === "/" && inputEl) inputEl.select();
  }

  let isTruncated = items.length > 20;
  let search;
  $: list = items
    .filter((item) => {
      if (search) {
        return item.title.toLowerCase().includes(search.toLowerCase()) || item.description.toLowerCase().includes(search.toLowerCase());
      }
      return true;
    })
    .slice(0, isTruncated ? 2 : items.length);
</script>

<svelte:head>
  <title>blog - {SITE_TITLE}</title>
  <meta name="description" content={`Latest ${SITE_TITLE} posts`} />
</svelte:head>

<svelte:window on:keyup={focusSearch} />

<section class="mx-auto mb-16 flex max-w-3xl flex-col items-start justify-center px-4 sm:px-8">
  <h1 class="mb-4 text-3xl font-bold tracking-tight text-black dark:text-white md:text-5xl">
    Blog
  </h1>
  <p class="my-8 text-gray-600 dark:text-gray-400">
		This is where you will find the things I write about that I've searched or tips that maybe useful
    in the future or for someone else. You can search below to filter, by title and description, 
    for things that interest you.<br/>
		I've written a total of {items.length} posts until now in this blog.
  </p>
  <div class="relative mb-4 w-full">
    <input
      aria-label="Search articles by title and description"
      type="text"
      bind:value={search}
      bind:this={inputEl}
      placeholder="Hit / to search by title and description"
      class="block w-full rounded-md border border-gray-200 bg-white px-4 py-2 text-gray-900 focus:border-yellow-500 focus:ring-yellow-500 dark:border-gray-900 dark:bg-gray-800 dark:text-gray-100"
    />
    <svg
      class="absolute right-3 top-3 h-5 w-5 text-gray-400 dark:text-gray-300"
      xmlns="http://www.w3.org/2000/svg"
      fill="none"
      viewBox="0 0 24 24"
      stroke="currentColor"
    >
      <path
        stroke-linecap="round"
        stroke-linejoin="round"
        stroke-width="2"
        d="M21 21l-6-6m2-5a7 7 0 11-14 0 7 7 0 0114 0z"
      />
    </svg
    >
  </div>
  {#if !search}
    <h2 class="mt-8 mb-4 text-2xl font-bold tracking-tight text-black dark:text-white md:text-4xl">
      All Posts
    </h2>
  {:else}
    <h2 class="mt-8 mb-4 text-2xl font-bold tracking-tight text-black dark:text-white md:text-4xl">
      Searching by {search}
    </h2>
  {/if}
  {#if list.length}
    <ul id="skip" class="w-full">
      {#each list as item}
        <li class="mb-8 text-lg">
          <IndexCard
            href={`/blog/${item.slug}`}
            title={item.title}
            stringData={dayjs(item.date).toISOString().slice(0, 10)}
          >
            {item.description}
          </IndexCard>
        </li>
      {/each}
    </ul>
    {#if isTruncated}
      <div class="flex justify-center">
        <button
          on:click={() => (isTruncated = false)}
          class="inline-block rounded bg-amber-200 p-4 text-lg font-bold tracking-tight text-black hover:bg-yellow-400 dark:bg-yellow-400 dark:text-white hover:dark:bg-yellow-500 md:text-2xl"
        >
          Load More Posts...
        </button>
      </div>
    {/if}
  {:else if search}
    <div class="prose dark:prose-invert">
      No posts found for
      <code>{search}</code>.
    </div>
    <button class="p-2 bg-slate-500" on:click={() => (search = '')}>Clear your search</button>
  {:else}
    <div class="prose dark:prose-invert">No blogposts found!</div>
  {/if}
</section>
