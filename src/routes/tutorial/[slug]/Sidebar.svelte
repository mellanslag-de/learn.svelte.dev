<script>
	import { createEventDispatcher } from 'svelte';
	import { afterNavigate } from '$app/navigation';
	import Modal from '$lib/components/Modal.svelte';
	import Menu from './Menu.svelte';

	/** @type {import('$lib/types').PartStub[]} */
	export let index;

	/** @type {import('$lib/types').Exercise} */
	export let exercise;

	const dispatch = createEventDispatcher();

	const namespace = 'learn.svelte.dev';
	const copy_enabled = `${namespace}:copy_enabled`;

	/** @type {HTMLElement} */
	let sidebar;

	let show_modal = false;

	afterNavigate(async () => {
		// TODO ideally we would associate scroll state with
		// history. That's a little tricky to do right now,
		// so for now just always reset sidebar scroll
		sidebar.scrollTop = 0;
	});
</script>

<Menu {index} current={exercise} />

<div
	bind:this={sidebar}
	class="text"
	on:copy={(e) => {
		if (sessionStorage[copy_enabled]) return;

		/** @type {HTMLElement | null} */
		let node = /** @type {HTMLElement} */ (e.target);

		while (node && node !== e.currentTarget) {
			if (node.nodeName === 'PRE') {
				show_modal = true;

				e.preventDefault();
				return;
			}

			node = /** @type {HTMLElement | null} */ (node.parentNode);
		}
	}}
>
	<!-- svelte-ignore a11y-click-events-have-key-events -->
	<div
		on:click={(e) => {
			const node = /** @type {HTMLElement} */ (e.target);

			if (node.nodeName === 'CODE') {
				const { file } = node.dataset;
				if (file) {
					dispatch('select', { file });
				}
			}
		}}
	>
		{@html exercise.html}
	</div>

	{#if exercise.next}
		<p><a href="/tutorial/{exercise.next.slug}">Next: {exercise.next.title}</a></p>
	{/if}
</div>

<footer>
	<a class="edit" href="https://github.com/sveltejs/learn.svelte.dev/tree/main/{exercise.dir}">
		Edit this page
	</a>
</footer>

{#if show_modal}
	<Modal on:close={() => (show_modal = false)}>
		<div class="modal-contents">
			<h2>Copy and paste is currently disabled!</h2>

			<p>
				We recommend typing the code into the editor to complete the exercise, as this results in
				better retention and understanding.
			</p>
			<label>
				<input
					type="checkbox"
					on:change={(e) => {
						sessionStorage[copy_enabled] = e.currentTarget.checked ? 'true' : '';
					}}
				/>
				enable copy-and-paste for the duration of this session
			</label>

			<button on:click={() => (show_modal = false)}>OK</button>
		</div>
	</Modal>
{/if}

<style>
	.text {
		flex: 1 1;
		overflow-y: auto;
		padding: 2.2rem 3rem;
		border-right: 1px solid var(--sk-back-4);
		background: var(--sk-back-3);
	}

	.text :global(pre) {
		background: var(--sk-back-1);
		box-shadow: inset 1px 1px 3px rgba(0, 0, 0, 0.1);
		border-radius: var(--sk-border-radius);
	}

	.text :global(pre) :global(.highlight) {
		--color: rgba(220, 220, 0, 0.2);
		background: var(--color);
		outline: 2px solid var(--color);
		border-radius: 2px;
	}

	.text :global(pre) :global(.highlight.add) {
		--color: rgba(0, 255, 0, 0.1);
	}

	.text :global(pre) :global(.highlight.remove) {
		--color: rgba(255, 0, 0, 0.1);
	}

	footer {
		padding: 1rem 3rem;
		display: flex;
		justify-content: space-between;
		background: var(--sk-back-3);
		border-top: 1px solid var(--sk-back-4);
		border-right: 1px solid var(--sk-back-4);
	}

	footer .edit {
		color: var(--sk-text-2);
		font-size: 1.4rem;
		padding: 0 0 0 1.4em;
		background: url($lib/icons/file-edit.svg) no-repeat 0 calc(50% - 0.1em);
		background-size: 1em 1em;
	}

	.modal-contents h2 {
		font-size: 2.4rem;
		margin: 0 0 0.5em 0;
	}

	.modal-contents label {
		user-select: none;
	}

	.modal-contents button {
		display: block;
		background: var(--prime);
		color: white;
		padding: 1rem;
		width: 10em;
		margin: 1em 0 0 0;
		border-radius: var(--sk-border-radius);
		line-height: 1;
	}
</style>
