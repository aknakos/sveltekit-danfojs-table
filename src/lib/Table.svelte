<script lang="ts">
	import type { SortByI } from './Table.js';

	import FilterBoxed from '$lib/icons/FilterBoxed.svelte';
	import SortDown from '$lib/icons/SortDown.svelte';
	import SortUp from '$lib/icons/SortUp.svelte';
	import { fade } from 'svelte/transition';
	import { Order } from './defs.js';

	export let df: any; // Danfo Dataframe

	export let sorting: boolean = true;
	export let sortBy: SortByI = { name: undefined, order: Order.ASC };

	export let filtering = true;
	export let filterBy: Record<string, any> = {};

	let hoveredHeading: string;
	const availableFilters: Record<string, Array<string>> = {
		string: ['contains', 'starts with', 'ends with'],
		float32: [
			'greater than',
			'less than',
			'greater than or equal',
			'less than or equal',
			'equals',
			'not equals'
		],
		int32: [
			'greater than',
			'less than',
			'greater than or equal',
			'less than or equal',
			'equals',
			'not equals'
		],
		boolean: ['equals']
	};
	const filtersymbols: Record<string, string> = {
		contains: '∈',
		'starts with': '∠',
		'ends with': '↵',
		equals: '≡',
		'not equals': '≠',
		'greater than': '>',
		'less than': '<',
		'greater than or equal': '≥',
		'less than or equal': '≤'
	};

	let _df: any; // Internal Dataframe for sorting/filter
	let recalculating;

	$: load(df);

	function load(df: any) {
		if (!df) return;

		_df = df.copy();
		filterBy = {};
		df.columns.map((column: string) => {
			filterBy[column] = {
				type: df[column].dtype,
				filter_type: availableFilters[df[column].dtype][0],
				by: ''
			};
		});
		// console.log(filterBy);
	}

	async function recaulculate() {
		recalculating = true;
		_df = df.copy();

		_df.columns.forEach((column: string) => {
			const by = filterBy[column].by;
			if (by && by !== '') {
				switch (filterBy[column].filter_type) {
					case 'contains':
						_df = _df.query(_df[column].str.toLowerCase().str.search(by.toLowerCase()).ge(0));
						break;
					case 'starts with':
						_df = _df.query(_df[column].str.toLowerCase().str.startsWith(by.toLowerCase()));
						break;
					case 'ends with':
						_df = _df.query(_df[column].str.toLowerCase().str.endsWith(by.toLowerCase()));
						break;
					case 'equals':
						_df = _df.query(_df[column].eq(parseFloat(by)));
						break;
					case 'not equals':
						_df = _df.query(_df[column].ne(parseFloat(by)));
						break;
					case 'greater than':
						_df = _df.query(_df[column].gt(parseFloat(by)));
						break;
					case 'less than':
						_df = _df.query(_df[column].lt(parseFloat(by)));
						break;
					case 'greater than or equal':
						_df = _df.query(_df[column].ge(parseFloat(by)));
						break;
					case 'less than or equal':
						_df = _df.query(_df[column].le(parseFloat(by)));
						break;
				}
			}
		});

		if (sortBy.name) _df = _df.sortValues(sortBy.name, { ascending: sortBy.order === 'ascending' });
		recalculating = false;
	}

	async function doSortBy(column: string) {
		if (!sorting) return;
		if (sortBy.name === column) {
			if (sortBy.order === Order.ASC) sortBy.order = Order.DESC;
			else {
				sortBy.order = Order.ASC;
				sortBy.name = undefined;
			}
		} else {
			sortBy.order = Order.ASC;
			sortBy.name = column;
		}
		recaulculate();
	}

	async function filterByCallback(column: string, text: string) {
		filterBy[column].by = text;
		recaulculate();
		// if (df.ctypes.at(column) !== 'string') text = parseFloat(text);
		// if (text && text !== '') filterBy[column] = text;
		// else delete filterBy[column];
		// recaulculate();
	}
</script>

{#if df}
	<table {...$$restProps}>
		<tr>
			<slot name="before-heading" />
			{#each _df.columns as column}
				<th class="cursor-pointer">
					<div class="flex items-center mx-2">
						<div class="w-full flex flex-col justify-center flex-grow-0">
							<span
								on:click={() => doSortBy(column)}
								class={'flex self-center justify-center items-center ' +
									(column === sortBy.name ? 'sort-column sort-' + sortBy.order : '')}
							>
								{column}
								{#if column === sortBy.name}
									{#if sortBy.order === 'ascending'}
										<SortUp />
									{:else}
										<SortDown />
									{/if}
								{/if}
							</span>
							<div class="h-full inline-flex self-center place-items-center items-center">
								<div class="dropdown">
									<button tabindex="0" class="btn btn-sm rounded-r-none">
										{filtersymbols[filterBy[column].filter_type]}
									</button>
									<ul
										tabindex="0"
										class="dropdown-content menu p-2 shadow bg-base-100 rounded-box w-52"
									>
										{#each availableFilters[filterBy[column].type] as item}
											<li>
												<button
													class="btn btn-ghost btn-sm text-left align-baseline justify-start"
													on:click={() => {
														filterBy[column].filter_type = item;
														filterBy[column].by = '';
													}}
												>
													{filtersymbols[item]}
													{item}
												</button>
											</li>
										{/each}
									</ul>
								</div>

								<input
									type={filterBy[column].type === 'string' ? 'text' : 'number'}
									class="w-full input input-sm input-bordered rounded-l-none"
									placeholder={`${filterBy[column].filter_type}:`}
									on:input={(event) => filterByCallback(column, event?.target?.value)}
									value={filterBy[column].by}
								/>
							</div>
							<slot name="heading" {column} />
						</div>
						{#if filtering && !filterBy.hasOwnProperty(column) && hoveredHeading === column}
							<span class="w-full">
								<FilterBoxed />
							</span>
						{/if}
					</div>
				</th>
			{/each}
			<slot name="after-heading" />
		</tr>
		<tbody>
			{#each _df.values as row, i (i)}
				<tr in:fade>
					<slot name="before" {row} index={i} />
					{#each row as value, ii (ii)}
						<td
							class={`${_df.columns[ii]} ${_df.columns[ii] === sortBy.name ? 'sort-column' : ''}`}
						>
							<slot
								name="value"
								{value}
								{row}
								row_index={i}
								col_index={ii}
								column={_df.columns[ii]}
							>
								{value}
							</slot>
						</td>
					{/each}
					<slot name="after" {row} index={i} />
				</tr>
			{/each}
		</tbody>
	</table>
{/if}

<style>
	.selection-col {
		transform: translate(50%, 0);
	}

	td {
		height: 1px;
	}
</style>
