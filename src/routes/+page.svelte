<script lang="ts">
	import { goto } from "$app/navigation";
	import { page } from "$app/stores";
	import { debounce } from "lodash";
	import { onDestroy } from "svelte";

	const params = {
		example: "",
		example2: "",
	};

	$: {
		const searchParams = $page.url.searchParams;
		params.example = searchParams.get("example") || "";
		params.example2 = searchParams.get("example2") || "";
	}

	// A function to update URL params. Removes unused params. Keeps the same order of params
	const updateSearchParams = (key: string, value: string) => {
		const previousSearchParams = $page.url.searchParams;
		const newSearchParams = new URLSearchParams(previousSearchParams);

		if (value === "") {
			newSearchParams.delete(key);
		} else {
			if (newSearchParams.get(key) !== value) {
				newSearchParams.set(key, value);
			}
		}

		newSearchParams.sort();

		const newSearchParamString = newSearchParams.toString();

		goto(`?${newSearchParamString}`, {
			replaceState: true, // Don't allow the back button to change on-page state
			noScroll: true, // Don't scroll to the top
			keepFocus: true, // Don't lose focus
		});
	};

	// Waits 250ms after a keyup and then updates the search params
	const handleKeyup = debounce((event: KeyboardEvent, key: string) => {
		const target = event.target as HTMLInputElement;

		if (target) {
			updateSearchParams(key, target.value);
		}
	}, 250);

	onDestroy(() => {
		/*
    Possibly not necessary here but just in case...
    The onDestroy lifecycle method is used here to make sure the debounce timer is cleaned up when this component is destroyed.
    */
		handleKeyup.cancel();
	});
</script>

<section>
	<label>
		Example
		<input
			type="text"
			value={params.example}
			on:keyup={(event) => {
				handleKeyup(event, "example");
			}}
		/>
	</label>

	<label>
		Example 2
		<input
			type="text"
			value={params.example2}
			on:keyup={(event) => {
				handleKeyup(event, "example2");
			}}
		/>
	</label>
</section>

<style>
	section {
		display: flex;
		flex-direction: column;
		row-gap: 3rem;
		padding: 1rem;
	}

	label {
		font-family: system-ui, -apple-system, BlinkMacSystemFont, "Segoe UI",
			Roboto, Oxygen, Ubuntu, Cantarell, "Open Sans", "Helvetica Neue",
			sans-serif;
		display: grid;
		grid-template-columns: 80px 200px;
		align-items: center;
		column-gap: 2rem;
	}

	input {
		padding: 0.5rem;
		border: 1px solid black;
		border-radius: 4px;
	}
</style>
