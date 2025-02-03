<script>
	import Footer from '$lib/components/Footer.svelte';

	let { children } = $props();
</script>

<main id="main-content" class="main-content">
	{@render children?.()}
</main>

<Footer />
