<script context="module">
  export const prerender = true; // you can uncomment to prerender as an optimization
  export const hydrate = true;

  export async function load({ params, fetch }) {
    const slug = params.slug;
    const { metadata, content } = await fetch(`/blog/${slug}.json`).then(r => r.json());
    return {
      props: { metadata, content, slug }
    };
  }
</script>

<script>
  import { MY_TWITTER_HANDLE } from "$lib/siteConfig";
  import dayjs from "dayjs";
  import { DEFAULT_OG_IMAGE } from "$lib/siteConfig.js";
  import { page } from "$app/stores";
  import Comments from "$lib/components/Comments.svelte";

  /** @type {import("$lib/types.js").ContentItem} */
  export let metadata, content; // warning: if you try to destructure content here, make sure to make it reactive, or your page content will not update when your user navigates
</script>

<svelte:head>
  <title>{metadata.title} - davidpalves</title>

  <link rel="canonical" href={$page.url} />
  <meta property="og:url" content={$page.url} />
  <meta property="og:type" content="article" />
  <meta property="og:title" content={metadata.title} />
  <meta name="Description" content={metadata.description} />
  <meta property="og:description" content={metadata.description} />
  <meta name="twitter:card" content={metadata.image ? 'summary_large_image' : 'summary'} />
  <meta name="twitter:creator" content={'@' + MY_TWITTER_HANDLE} />
  <meta name="twitter:title" content={metadata.title} />
  <meta name="twitter:description" content={metadata.description} />
  {#if metadata.image}
    <meta property="og:image" content={metadata.image} />
    <meta name="twitter:image" content={metadata.image} />
  {:else}
    <meta property="og:image" content={DEFAULT_OG_IMAGE} />
    <meta name="twitter:image" content={DEFAULT_OG_IMAGE} />
  {/if}
</svelte:head>

<article
  id="skip"
  class="mx-auto mb-8 flex w-full max-w-3xl flex-col items-start justify-center sm:px-8"
>
  <h1 class="mb-8 text-3xl font-bold tracking-tight text-black dark:text-white md:text-5xl ">
    {metadata.title}
  </h1>
  <div
    class="bg mt-2 flex w-full justify-between sm:flex-col sm:items-start md:flex-row md:items-center"
  >
  <p class="min-w-32 flex items-center text-sm text-gray-600 dark:text-gray-400 md:mt-0">
    {dayjs(metadata.date).toISOString().slice(0, 10)}
  </p>
  <p class="flex items-center text-sm text-gray-700 dark:text-gray-300">David Pierre</p>
  </div>
  <div
    class="my-2 flex h-1 bg-gradient-to-r from-yellow-400 via-amber-500 to-orange-400 sm:mx-0 w-full"></div>

  <div class="prose mt-12 w-full max-w-none dark:prose-invert">
    {@html content}
  </div>

  <Comments />
</article>
