<script lang="ts">
	type Player = { name: string; power: number };

	let rawPlayers = $state('');
	let rawSubs = $state('');

	let output = $state('');

	let newPlayerName = $state('');
	let newPlayerPower = $state<number | null>(null);
	let newSubName = $state('');
	let newSubPower = $state<number | null>(null);

	let players = $derived(parsePlayers(rawPlayers));
	let subs = $derived(parsePlayers(rawSubs));

	function parsePlayers(input: string): Player[] {
		return input
			.split(/\r?\n/)
			.map((l) => l.trim())
			.filter(Boolean)
			.map((line) => {
				const m = line.match(/^([^:\-\t]+)[\:\-\t]\s*(\d+)\s*$/);
				if (!m) return null;
				const name = m[1].trim();
				if (!name) return null;
				return { name, power: Number(m[2]) } as Player;
			})
			.filter((x): x is Player => !!x)
			.sort((a, b) => b.power - a.power)
			.slice(0, 30);
	}

	function fmtNames(arr: Player[] | string[]): string {
		return arr.map((x: any) => (typeof x === 'string' ? x : x.name)).join(', ');
	}

	function formatPlayers(list: Player[]): string {
		return list.map((p) => `${p.name}: ${p.power}`).join('\n');
	}

	function formatSubs(list: Player[]): string {
		return list.map((p) => `${p.name}: ${p.power}`).join('\n');
	}

	function hasValidPower(value: number | null): value is number {
		return value !== null && !Number.isNaN(value);
	}

	function addPlayer() {
		const trimmedName = newPlayerName.trim();

		if (!trimmedName || !hasValidPower(newPlayerPower)) {
			return;
		}

		if (players.length >= 30) {
			return;
		}

		const sanitizedPower = Math.max(0, Math.round(newPlayerPower));
		const updated = [...players, { name: trimmedName, power: sanitizedPower }].sort(
			(a, b) => b.power - a.power
		);

		rawPlayers = formatPlayers(updated);
		newPlayerName = '';
		newPlayerPower = null;
	}

	function removePlayer(index: number) {
		const updated = players.filter((_, i) => i !== index);
		rawPlayers = formatPlayers(updated);
	}

	function addSub() {
		const trimmed = newSubName.trim();

		if (!trimmed || !hasValidPower(newSubPower)) {
			return;
		}

		if (subs.length >= 10) {
			return;
		}

		const sanitizedPower = Math.max(0, Math.round(newSubPower));
		const updated = [...subs, { name: trimmed, power: sanitizedPower }].sort(
			(a, b) => b.power - a.power
		);

		rawSubs = formatSubs(updated);
		newSubName = '';
		newSubPower = null;
	}

	function removeSub(index: number) {
		const updated = subs.filter((_, i) => i !== index);
		rawSubs = formatSubs(updated);
	}

	function generate() {
		const orderedPlayers = [...players];
		const totalPlayers = orderedPlayers.length;

		// Automatically distribute: ~30% attackers, ~35% support, ~35% defenders
		const attackerCount = Math.round(totalPlayers * 0.3);
		const supportCount = Math.round(totalPlayers * 0.35);

		const attackers = orderedPlayers.slice(0, attackerCount);
		const supporters = orderedPlayers.slice(attackerCount, attackerCount + supportCount);
		const defenders = orderedPlayers.slice(attackerCount + supportCount);

		output = `🛡️ Swordland Showdown Teams

Attackers:
${fmtNames(attackers)}

Support:
${fmtNames(supporters)}

Defenders:
${fmtNames(defenders)}

Subs:
${fmtNames(subs)}`;
	}

	async function copyOut() {
		await navigator.clipboard.writeText(output);
	}
</script>

<svelte:head>
	<title>Swordland Showdown - Team Generator</title>
</svelte:head>

<h1 class="text-3xl font-semibold tracking-tight text-slate-900 dark:text-white">
	Swordland Showdown - Team Generator
</h1>

<section class="mt-6 grid w-full max-w-4xl gap-6">
	<div class="space-y-2">
		<label for="players-input" class="block text-sm font-medium text-slate-700 dark:text-slate-200">
			Players (Name: Power per line)
		</label>
		<textarea
			id="players-input"
			rows="12"
			bind:value={rawPlayers}
			class="w-full rounded-lg border border-slate-300 bg-white px-3 py-2 text-sm text-slate-900 shadow-sm transition placeholder:text-slate-400 focus:border-blue-500 focus:ring-2 focus:ring-blue-500/40 focus:outline-none dark:border-slate-700 dark:bg-slate-900 dark:text-slate-100"
		></textarea>
		<p class="text-xs text-slate-500 dark:text-slate-400">
			Paste each player on a new line using the format <span class="font-medium">Name: Power</span>.
			Invalid lines will be ignored. Max 30 players.
		</p>
	</div>

	<div
		class="space-y-4 rounded-2xl border border-slate-200 bg-white p-4 shadow-sm dark:border-slate-700 dark:bg-slate-900"
	>
		<form
			onsubmit={(e) => {
				e.preventDefault();
				addPlayer();
			}}
			class="flex flex-wrap items-end gap-4"
		>
			<div class="min-w-[220px] flex-1 space-y-2">
				<label
					for="player-name"
					class="block text-sm font-medium text-slate-700 dark:text-slate-200"
				>
					Add player
				</label>
				<input
					id="player-name"
					type="text"
					placeholder="Player name"
					bind:value={newPlayerName}
					class="w-full rounded-lg border border-slate-300 px-3 py-2 text-sm text-slate-900 transition placeholder:text-slate-400 focus:border-blue-500 focus:ring-2 focus:ring-blue-500/40 focus:outline-none dark:border-slate-700 dark:bg-slate-800 dark:text-slate-100"
				/>
			</div>
			<div class="space-y-2">
				<label
					for="player-power"
					class="block text-sm font-medium text-slate-700 dark:text-slate-200"
				>
					Power
				</label>
				<input
					id="player-power"
					type="number"
					min="0"
					step="1"
					bind:value={newPlayerPower}
					class="w-24 rounded-lg border border-slate-300 px-3 py-2 text-sm text-slate-900 transition focus:border-blue-500 focus:ring-2 focus:ring-blue-500/40 focus:outline-none dark:border-slate-700 dark:bg-slate-800 dark:text-slate-100"
				/>
			</div>
			<button
				type="submit"
				class="inline-flex items-center rounded-lg bg-blue-600 px-4 py-2 text-sm font-semibold text-white shadow-sm transition hover:bg-blue-500 focus:outline-none focus-visible:ring-2 focus-visible:ring-blue-500 focus-visible:ring-offset-2 disabled:cursor-not-allowed disabled:bg-blue-300"
				disabled={!newPlayerName.trim() || !hasValidPower(newPlayerPower) || players.length >= 30}
			>
				Add player ({players.length}/30)
			</button>
		</form>

		{#if players.length}
			<div class="overflow-hidden rounded-xl border border-slate-200 dark:border-slate-700">
				<table class="w-full text-sm text-slate-700 dark:text-slate-200">
					<thead
						class="bg-slate-50 text-xs font-semibold tracking-wide text-slate-500 uppercase dark:bg-slate-800 dark:text-slate-400"
					>
						<tr>
							<th class="px-4 py-3 text-left">Name</th>
							<th class="w-24 px-4 py-3 text-right">Power</th>
							<th class="w-28 px-4 py-3 text-right"></th>
						</tr>
					</thead>
					<tbody class="divide-y divide-slate-200 dark:divide-slate-700">
						{#each players as player, index}
							<tr class="bg-white dark:bg-slate-900">
								<td class="px-4 py-3 font-medium text-slate-900 dark:text-slate-100"
									>{player.name}</td
								>
								<td class="px-4 py-3 text-right text-slate-600 dark:text-slate-300"
									>{player.power}</td
								>
								<td class="px-4 py-3 text-right">
									<button
										type="button"
										class="text-sm font-semibold text-red-500 transition hover:text-red-400 focus:outline-none"
										onclick={() => removePlayer(index)}
									>
										Remove
									</button>
								</td>
							</tr>
						{/each}
					</tbody>
				</table>
			</div>
		{:else}
			<p
				class="rounded-lg bg-slate-50 px-4 py-3 text-sm text-slate-500 dark:bg-slate-800 dark:text-slate-400"
			>
				No players yet — add one above or paste into the textarea.
			</p>
		{/if}
	</div>

	<div class="space-y-2">
		<label for="subs-input" class="block text-sm font-medium text-slate-700 dark:text-slate-200">
			Subs (Name per line, optional)
		</label>
		<textarea
			id="subs-input"
			rows="6"
			bind:value={rawSubs}
			class="w-full rounded-lg border border-slate-300 bg-white px-3 py-2 text-sm text-slate-900 shadow-sm transition placeholder:text-slate-400 focus:border-blue-500 focus:ring-2 focus:ring-blue-500/40 focus:outline-none dark:border-slate-700 dark:bg-slate-900 dark:text-slate-100"
		></textarea>
	</div>

	<div
		class="space-y-4 rounded-2xl border border-slate-200 bg-white p-4 shadow-sm dark:border-slate-700 dark:bg-slate-900"
	>
		<form
			onsubmit={(e) => {
				e.preventDefault();
				addSub();
			}}
			class="flex flex-wrap items-end gap-4"
		>
			<div class="min-w-[220px] flex-1 space-y-2">
				<label for="sub-name" class="block text-sm font-medium text-slate-700 dark:text-slate-200">
					Add sub
				</label>
				<input
					id="sub-name"
					type="text"
					placeholder="Sub name"
					bind:value={newSubName}
					class="w-full rounded-lg border border-slate-300 px-3 py-2 text-sm text-slate-900 transition placeholder:text-slate-400 focus:border-blue-500 focus:ring-2 focus:ring-blue-500/40 focus:outline-none dark:border-slate-700 dark:bg-slate-800 dark:text-slate-100"
				/>
			</div>
			<div class="space-y-2">
				<label for="sub-power" class="block text-sm font-medium text-slate-700 dark:text-slate-200">
					Power
				</label>
				<input
					id="sub-power"
					type="number"
					min="0"
					step="1"
					bind:value={newSubPower}
					class="w-24 rounded-lg border border-slate-300 px-3 py-2 text-sm text-slate-900 transition focus:border-blue-500 focus:ring-2 focus:ring-blue-500/40 focus:outline-none dark:border-slate-700 dark:bg-slate-800 dark:text-slate-100"
				/>
			</div>
			<button
				type="submit"
				class="inline-flex items-center rounded-lg bg-blue-600 px-4 py-2 text-sm font-semibold text-white shadow-sm transition hover:bg-blue-500 focus:outline-none focus-visible:ring-2 focus-visible:ring-blue-500 focus-visible:ring-offset-2 disabled:cursor-not-allowed disabled:bg-blue-300"
				disabled={!newSubName.trim() || !hasValidPower(newSubPower) || subs.length >= 10}
			>
				Add sub ({subs.length}/10)
			</button>
		</form>

		{#if subs.length}
			<div class="overflow-hidden rounded-xl border border-slate-200 dark:border-slate-700">
				<table class="w-full text-sm text-slate-700 dark:text-slate-200">
					<thead
						class="bg-slate-50 text-xs font-semibold tracking-wide text-slate-500 uppercase dark:bg-slate-800 dark:text-slate-400"
					>
						<tr>
							<th class="px-4 py-3 text-left">Name</th>
							<th class="w-24 px-4 py-3 text-right">Power</th>
							<th class="w-28 px-4 py-3 text-right"></th>
						</tr>
					</thead>
					<tbody class="divide-y divide-slate-200 dark:divide-slate-700">
						{#each subs as sub, index}
							<tr class="bg-white dark:bg-slate-900">
								<td class="px-4 py-3 font-medium text-slate-900 dark:text-slate-100">{sub.name}</td>
								<td class="px-4 py-3 text-right text-slate-600 dark:text-slate-300">{sub.power}</td>
								<td class="px-4 py-3 text-right">
									<button
										type="button"
										class="text-sm font-semibold text-red-500 transition hover:text-red-400 focus:outline-none"
										onclick={() => removeSub(index)}
									>
										Remove
									</button>
								</td>
							</tr>
						{/each}
					</tbody>
				</table>
			</div>
		{:else}
			<p
				class="rounded-lg bg-slate-50 px-4 py-3 text-sm text-slate-500 dark:bg-slate-800 dark:text-slate-400"
			>
				No subs added yet.
			</p>
		{/if}
	</div>

	<div class="space-y-3">
		<div class="space-y-2">
			<label for="output-text" class="block text-sm font-medium text-slate-700 dark:text-slate-200">
				Output (plain text + emojis)
			</label>
			<textarea
				id="output-text"
				rows="10"
				readonly
				bind:value={output}
				class="w-full rounded-lg border border-slate-300 bg-slate-50 px-3 py-2 text-sm text-slate-900 shadow-inner focus:outline-none dark:border-slate-700 dark:bg-slate-900 dark:text-slate-100"
			></textarea>
		</div>
		<div class="flex gap-3">
			<button
				class="inline-flex items-center rounded-lg bg-emerald-600 px-4 py-2 text-sm font-semibold text-white shadow-sm transition hover:bg-emerald-500 focus:outline-none focus-visible:ring-2 focus-visible:ring-emerald-500 focus-visible:ring-offset-2"
				onclick={generate}
			>
				Regenerate
			</button>
			<button
				class="inline-flex items-center rounded-lg bg-slate-800 px-4 py-2 text-sm font-semibold text-white shadow-sm transition hover:bg-slate-700 focus:outline-none focus-visible:ring-2 focus-visible:ring-slate-500 focus-visible:ring-offset-2"
				onclick={copyOut}
			>
				Copy
			</button>
		</div>
	</div>
</section>
