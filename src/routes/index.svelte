<script>
	import Table from '$lib/Table.svelte';
	import Danfojs from '@aknakos/sveltekit-danfojs/Danfojs.svelte';
	import { dfd } from '@aknakos/sveltekit-danfojs';

	let df;

	$: if ($dfd) df = new $dfd.DataFrame({ a: [1, 2, 3, 4, 5], b: [1, 2, 3, 4, 5] });
</script>

<Danfojs>
	{#if df}
		<Table {df} />
	{/if}
</Danfojs>
