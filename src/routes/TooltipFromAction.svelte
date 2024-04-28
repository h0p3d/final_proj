<script>
    //Source: https://svelte.dev/repl/dd6754a2ad0547c5b1c1ea37c0293fef?version=4.2.15
	export let title;
	export let x;
	export let y;
</script>
<div style="top: {y + 5}px; left: {x + 5}px;">
	{@html title}
</div>

<style>
	div {
		border: 1px solid #ddd;
		box-shadow: 1px 1px 1px #ddd;
		background: white;
		border-radius: 4px;
		padding: 4px;
		position: absolute;
	}
</style>
