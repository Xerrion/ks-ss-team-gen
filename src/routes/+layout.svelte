<script lang="ts">
	import '../app.css';
	import favicon from '$lib/assets/favicon.svg';

	let { children } = $props();
</script>

<svelte:head>
	<link rel="icon" href={favicon} />
</svelte:head>

<div class="min-h-screen bg-slate-950 text-slate-100 antialiased">
	<main class="mx-auto flex w-full max-w-5xl flex-col gap-10 px-4 py-10">
		{@render children()}
	</main>
</div>
