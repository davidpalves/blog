<script context="module">
  import { MY_LINKEDIN, MY_TWITTER_HANDLE, REPO_URL, SITE_TITLE } from "$lib/siteConfig";
</script>

<script>
  import "../tailwind.css";
  import "$lib/hljs-github-dark.css";
  import Nav from "$lib/components/Nav.svelte";
  import { navigationIsDelayed } from "$lib/stores.js";
  import { fade } from "svelte/transition";
  import { Circle3 } from "svelte-loading-spinners";
</script>

<svelte:head>
  <link
    rel="alternate"
    type="application/rss+xml"
    title={'RSS Feed for ' + SITE_TITLE}
    href="/feed.xml"
  />
</svelte:head>

{#if $navigationIsDelayed}
  <div class="fixed w-full h-full z-10" in:fade={{ duration: 150 }}>
    <div class="absolute w-full h-full bg-white dark:bg-amber-500 opacity-50 z-10"></div>
    <div class="absolute w-full h-full flex justify-center items-center z-20">
      <Circle3 />
    </div>
  </div>
{/if}

<div class="flex flex-col justify-center bg-gray-50 px-4 dark:bg-gray-900 sm:px-8">
  <Nav />
</div>
<main class="flex flex-col justify-center bg-gray-50 px-4 dark:bg-gray-900 sm:px-8">
  <slot />
</main>

<footer class="mx-auto mb-8 flex w-full max-w-3xl flex-col items-start justify-center">
  <hr class="border-1 mb-8 w-full border-gray-200 dark:border-gray-800" />
  <div class="grid w-full max-w-3xl grid-cols-1 gap-4 px-4 pb-16 sm:grid-cols-2 sm:px-8">
    <div class="flex flex-col space-y-4">
      <a class="text-gray-500 transition hover:text-gray-300" href="/">Home</a>
      <a class="text-gray-500 transition hover:text-gray-300" href="/about">About me</a>
      <a class="text-gray-500 transition hover:text-gray-300" href="/feed.xml" rel="external">
        RSS
      </a>
    </div>
    <div class="flex flex-col space-y-4">
      <a
        class="text-gray-500 transition hover:text-gray-300"
        target="_blank"
        rel="noopener noreferrer"
        href={'https://twitter.com/intent/follow?screen_name=' + MY_TWITTER_HANDLE}
      >
        Twitter
      </a>
      <a
        class="text-gray-500 transition hover:text-gray-300"
        target="_blank"
        rel="noopener noreferrer"
        href={REPO_URL}
      >
        GitHub
      </a>
      <a
        class="text-gray-500 transition hover:text-gray-300"
        target="_blank"
        rel="noopener noreferrer"
        href={MY_LINKEDIN}
      >
        Linkedin
      </a>
    </div>
  </div>
  <p class="prose px-4 dark:prose-invert sm:px-8">
    This blog is based on the
    <a href="https://swyxkit.netlify.app/">swyxkit</a>
    template, heavily modified.
  </p>
</footer>
