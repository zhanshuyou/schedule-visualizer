<script lang="ts">
	import '../app.css';
	import favicon from '$lib/assets/favicon.svg';

	let { children } = $props();
</script>

<svelte:head>
	<link rel="icon" href={favicon} />

	<meta name="google-adsense-account" content="ca-pub-7019817095267700" />
	<script
		async
		src="https://pagead2.googlesyndication.com/pagead/js/adsbygoogle.js?client=ca-pub-7019817095267700"
		crossorigin="anonymous"
	></script>
</svelte:head>

{@render children?.()}
