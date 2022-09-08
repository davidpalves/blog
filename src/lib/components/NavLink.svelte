<script>
	import { page } from '$app/stores';
	$: isActive = $page.url.pathname === $$props.href;
	export let href;
</script>

<a
	class="hidden rounded-lg p-1 text-gray-800 transition-all hover:bg-amber-300 hover:text-neutral-900 hover:underline dark:text-yellow-400 dark:hover:text-neutral-900 dark:hover:bg-amber-400 sm:px-3 sm:py-2 md:inline-block"
	class:font-semibold={isActive}
	{href}
	><span class="capsize"><slot /> </span>
</a>
