<script lang="ts">
	import { getModalStore, type ModalStore } from './stores';
	import { m } from '$paraglide/messages';

	interface Action {
		label: string;
		action: () => boolean | Promise<boolean>;
		async: boolean;
		classes?: string;
		btnIcon?: string;
		description?: string;
	}

	interface Props {
		/** Exposes parent props to this component. */
		parent: any;
		actions: Action[];
	}

	let { parent, actions }: Props = $props();

	const modalStore: ModalStore = getModalStore();

	let loadingAction: number | null = $state(null);

	async function handleAction(action: Action, index: number, event: MouseEvent) {
		if (loadingAction !== null) return;
		if (action.async) {
			loadingAction = index;
		}
		try {
			const result = await action.action();
			if (result) parent.onConfirm(event);
		} finally {
			loadingAction = null;
		}
	}
</script>

{#if $modalStore[0]}
	<div
		class="relative overflow-hidden rounded-2xl bg-white shadow-2xl"
		style="width: min(540px, 92vw);"
	>
		<!-- Decorative top band -->
		<div class="h-2 bg-gradient-to-r from-sky-600 via-cyan-500 to-sky-400"></div>

		<div class="px-8 pt-8 pb-6">
			<!-- Header area -->
			<div class="flex items-start gap-4 mb-6">
				<div
					class="flex-shrink-0 w-12 h-12 rounded-xl bg-gradient-to-br from-sky-600 to-cyan-500 flex items-center justify-center shadow-lg shadow-sky-200"
				>
					<i class="fa-solid fa-shield-halved text-white text-xl"></i>
				</div>
				<div class="min-w-0">
					<h2 class="text-xl font-bold text-gray-900 leading-tight">
						{$modalStore[0].title ?? m.firstTimeLoginModalTitle()}
					</h2>
					<p class="text-sm text-gray-500 mt-1">
						{m.firstTimeLoginModalDescription()}
					</p>
				</div>
			</div>

			<!-- Action cards -->
			<div class="space-y-3">
				{#each actions as action, i}
					{@const isPreset = i === 0}
					{@const isLoading = loadingAction === i}
					<button
						data-testid="first-login-action-{i}"
						onclick={(e) => handleAction(action, i, e)}
						disabled={loadingAction !== null}
						class="group w-full text-left rounded-xl border-2 p-4 transition-all duration-200
							{isPreset
							? 'border-sky-200 bg-gradient-to-r from-sky-50 to-cyan-50 hover:border-sky-400 hover:shadow-md hover:shadow-sky-100'
							: 'border-gray-150 bg-white hover:border-gray-300 hover:bg-gray-50'}
							{loadingAction !== null && !isLoading ? 'opacity-50 cursor-not-allowed' : 'cursor-pointer'}"
					>
						<div class="flex items-center gap-4">
							<div
								class="flex-shrink-0 w-10 h-10 rounded-lg flex items-center justify-center transition-transform duration-200 group-hover:scale-110
								{isPreset
									? 'bg-gradient-to-br from-sky-600 to-cyan-500 text-white shadow-md shadow-sky-200'
									: i === 1
										? 'bg-amber-100 text-amber-600'
										: 'bg-gray-100 text-gray-500'}"
							>
								{#if isLoading}
									<i class="fa-solid fa-spinner fa-spin text-lg"></i>
								{:else if action.btnIcon}
									<i class="fa-solid {action.btnIcon} text-lg"></i>
								{/if}
							</div>
							<div class="flex-1 min-w-0">
								<div
									class="font-semibold text-sm leading-tight
									{isPreset ? 'text-sky-900' : 'text-gray-800'}"
								>
									{action.label}
								</div>
								{#if action.description}
									<div class="text-xs text-gray-500 mt-0.5 leading-relaxed">
										{action.description}
									</div>
								{/if}
							</div>
							<i
								class="fa-solid fa-chevron-right text-xs transition-transform duration-200 group-hover:translate-x-0.5
								{isPreset ? 'text-sky-400' : 'text-gray-300'}"
							></i>
						</div>
						{#if isPreset}
							<div class="mt-2 ml-14">
								<span
									class="inline-block text-[10px] font-semibold tracking-wide uppercase text-sky-600 bg-sky-100 px-2 py-0.5 rounded-full"
								>
									{m.recommended()}
								</span>
							</div>
						{/if}
					</button>
				{/each}
			</div>
		</div>

		<!-- Footer -->
		<div class="px-8 pb-6 pt-2 flex justify-end">
			<button
				onclick={parent.onClose}
				class="text-sm text-gray-400 hover:text-gray-600 transition-colors cursor-pointer"
			>
				{m.skipForNow()}
			</button>
		</div>
	</div>
{/if}
