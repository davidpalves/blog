<script context="module">
  import { DEFAULT_OG_IMAGE, MY_TWITTER_HANDLE, SITE_DESCRIPTION, SITE_TITLE, SITE_URL, REPO_URL, MY_LINKEDIN } from "$lib/siteConfig";

  export const prerender = true; // index page is most visited, lets prerender

  export async function load({ fetch }) {
    const res = await fetch(`/api/listContent.json`);
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
  import FeatureCard from "$lib/components/FeatureCard.svelte";

  export let items;
</script>

<svelte:head>
  <title>home! - {SITE_TITLE}</title>
  <link rel="canonical" href={SITE_URL} />
  <link rel="alternate" type="application/rss+xml" href={SITE_URL + '/feed.xml'} />
  <meta property="og:url" content={SITE_URL} />
  <meta property="og:type" content="article" />
  <meta property="og:title" content={SITE_TITLE} />
  <meta name="Description" content={SITE_DESCRIPTION} />
  <meta property="og:description" content={SITE_DESCRIPTION} />
  <meta property="og:image" content={DEFAULT_OG_IMAGE} />
  <meta name="twitter:card" content="summary" />
  <meta name="twitter:creator" content={'@' + MY_TWITTER_HANDLE} />
  <meta name="twitter:title" content={SITE_TITLE} />
  <meta name="twitter:description" content={SITE_DESCRIPTION} />
  <meta name="twitter:image" content={DEFAULT_OG_IMAGE} />
</svelte:head>

<div
  class="mx-auto flex max-w-3xl flex-col items-start justify-center border-gray-200 px-4 pb-16 dark:border-gray-700 sm:px-8"
>
  <div class="flex flex-col-reverse items-start sm:flex-row mb-8">
    <div class="flex flex-col pr-8">
      <h1 class="mb-8 text-2xl sm:text-3xl font-bold tracking-tight text-black dark:text-white md:text-5xl">
        Hi there! I'm
        <span
          class="relative ml-2 inline-block before:absolute before:-inset-1 before:block before:-skew-y-3 before:bg-amber-400"
        >
					<span class="relative skew-y-3 text-neutral-900 dark:text-neutral-100">David</span>
				</span>
        !
        <img src="/memoji.png" class="relative inline-block" width="85px" alt="An Avatar of David's face">
      </h1>
      <h2 class="mb-4 text-gray-700 dark:text-gray-200">
				I'm a Brazilian software engineer, passionate about open source and cyber security.<br/>
				Here I'll try to share some learnings, tips, random thoughts, and pretty much anything I feel like sharing.
      </h2>
      <p class="mb-4 text-gray-700 dark:text-white">
        Feel free to get in touch with me through the links below!
      </p>
      <div class="flex flex-col-reverse items-center text-gray-700 dark:text-white">
        <ul class="space-x-20 content-center space-y-5">
          <li class="inline-block">
            <a 
              class="text-gray-500 transition hover:text-gray-300"
              target="_blank"
              rel="noopener noreferrer"
              href={REPO_URL}>
                <i class="fa fa-github"></i>
            </a>
          </li>
          <li class="inline-block">
            <a 
              class="text-gray-500 transition hover:text-gray-300"
              target="_blank"
              rel="noopener noreferrer"
              href="{MY_LINKEDIN}">
                <i class="fa fa-linkedin"></i>
            </a>
          </li>
          <li class="inline-block">
            <a 
              class="text-gray-500 transition hover:text-gray-300"
              target="_blank"
              rel="noopener noreferrer"
              href={'https://twitter.com/intent/follow?screen_name=' + MY_TWITTER_HANDLE}>
              <i class="fa fa-twitter"></i>
            </a>
          </li>
          <li class="inline-block">
            <a 
              class="text-gray-500 transition hover:text-gray-300"
              target="_blank"
              rel="noopener noreferrer"
              href="mailto:davidpierrealves21@gmail.com">
              <i class="fa fa-envelope"></i>
            </a>
          </li>
        </ul>
      </div>
    </div>
  </div>

  <section id="skip" class="mb-16 w-full">
    <h3 class="mb-6 text-2xl font-bold tracking-tight text-black dark:text-white md:text-4xl">
      Recent Posts
    </h3>
    <div class="flex flex-col gap-6 md:flex-row">
      {#each items as item, i}
        {#if i < 3}
          <FeatureCard title={item.title} href={`/blog/${item.slug}`} stringData={item.date} />
        {/if}
      {/each}
    </div>
    <a
      class="mt-8 flex h-6 rounded-lg leading-7 text-gray-600 transition-all dark:text-gray-400 dark:hover:text-gray-200"
      href="/blog"
    >See more posts
      <svg
        xmlns="http://www.w3.org/2000/svg"
        fill="none"
        viewBox="0 0 24 24"
        class="ml-1 h-6 w-6"
      >
        <path
          stroke="currentColor"
          stroke-linecap="round"
          stroke-linejoin="round"
          stroke-width="2"
          d="M17.5 12h-15m11.667-4l3.333 4-3.333-4zm3.333 4l-3.333 4 3.333-4z"
        />
      </svg
      >
    </a
    >
  </section>
</div>
