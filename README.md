# Simple table for danfojs dataframes

Example of how to construct a table based on a DanfoJS dataframe. This module uses tailwind and daisy ui's classes, so it will work with them installed or you can redefine the same classes as they do to customize the table. This module is mostly to be used as a way to prototype your own table.

Simple Usage:

```javascript
<script>
	import Table from '@aknakos/sveltekit-danfojs-table/Table.svelte';

	// Using danfojs importing from another library which you may also need..
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
```

You can also customize what is shown within the table using slot components:

| slot name      | exposed variables with the let: directive                                                                                                                                                     | Description                                                                                                                                                                      |
|----------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| before-heading | none                                                                                                                                                                                          | Something you may want to put before the variable heading entries. Its within the heading row, so it is a way to include extra rows with custom values, buttons, checkboxes, etc |
| heading        | let:column                                                                                                                                                                                    | Any extra information you may wish to include below each of the column's heading                                                                                                 |
| after-heading  | none                                                                                                                                                                                          | Same as before-heading but after.                                                                                                                                                |
| before         | let:row<br>let:index                                                                                                                                                                          | If you have extra (custom) columns created with the before-heading, here you can specify what goes in each of the rows                                                           |
| value          | let:value => the actual value of the cell<br>let:row => entire row as an array<br>let:row_index => index of row<br>let:col_index => index of column<br>let:column => name of column as string | By default the value is shown here. You can override to include color, badges, text-aligments, etc.                                                                              |
| after          | let:row<br>let:index                                                                                                                                                                          | Same as "before" but for custom columns within the after-heading.                                                                                                                |                                                                                                            |
