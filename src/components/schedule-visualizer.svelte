<script lang="ts">
	import { onMount } from 'svelte';
	import { m } from '../paraglide/messages';
	import { getLocale, setLocale } from '../paraglide/runtime';

	let mode = 'cron'; // 'cron' or 'rrule'
	let copied = false;

	// Cron state
	let cronMinute = '*';
	let cronHour = '*';
	let cronDay = '*';
	let cronMonth = '*';
	let cronWeekday = '*';

	// RRule state
	let rruleFreq = 'DAILY';
	let rruleInterval = '1';
	let rruleCount = '';
	let rruleUntil = '';
	let rruleByDay: string[] = [];
	let rruleByMonthDay = '';

	// Generate cron expression
	$: cronExpression = `${cronMinute} ${cronHour} ${cronDay} ${cronMonth} ${cronWeekday}`;

	// Generate RRule expression
	$: rruleExpression = (() => {
		let parts = [`FREQ=${rruleFreq}`];

		if (rruleInterval && rruleInterval !== '1') {
			parts.push(`INTERVAL=${rruleInterval}`);
		}

		if (rruleCount) {
			parts.push(`COUNT=${rruleCount}`);
		}

		if (rruleUntil) {
			const date = new Date(rruleUntil);
			const formatted = date.toISOString().replace(/[-:]/g, '').split('.')[0] + 'Z';
			parts.push(`UNTIL=${formatted}`);
		}

		if (rruleByDay.length > 0) {
			parts.push(`BYDAY=${rruleByDay.join(',')}`);
		}

		if (rruleByMonthDay) {
			parts.push(`BYMONTHDAY=${rruleByMonthDay}`);
		}

		return parts.join(';');
	})();

	// Human readable descriptions
	$: cronDescription = (() => {
		const parts = [];

		if (
			cronMinute === '*' &&
			cronHour === '*' &&
			cronDay === '*' &&
			cronMonth === '*' &&
			cronWeekday === '*'
		) {
			return m.cron_execute_minute();
		}

		if (cronMinute !== '*') {
			if (cronMinute.includes('/')) {
				const interval = cronMinute.split('/')[1];
				parts.push(m.cron_minute_interval({ interval }));
			} else if (cronMinute.includes(',')) {
				parts.push(m.cron_minute_specific({ minute: cronMinute }));
			} else {
				parts.push(m.cron_minute_specific({ minute: cronMinute }));
			}
		}

		if (cronHour !== '*') {
			if (cronHour.includes('/')) {
				const interval = cronHour.split('/')[1];
				parts.push(m.cron_hour_interval({ interval }));
			} else {
				parts.push(m.cron_hour_specific({ hour: cronHour }));
			}
		}

		if (cronDay !== '*') {
			parts.push(m.cron_day_specific({ day: cronDay }));
		}

		if (cronMonth !== '*') {
			const months = [
				m.month_1(),
				m.month_2(),
				m.month_3(),
				m.month_4(),
				m.month_5(),
				m.month_6(),
				m.month_7(),
				m.month_8(),
				m.month_9(),
				m.month_10(),
				m.month_11(),
				m.month_12()
			];
			if (cronMonth.includes(',')) {
				const monthNums = cronMonth.split(',');
				parts.push(monthNums.map((m) => months[parseInt(m) - 1]).join('、'));
			} else {
				parts.push(months[parseInt(cronMonth) - 1]);
			}
		}

		if (cronWeekday !== '*') {
			const days = [
				m.weekday_0(),
				m.weekday_1(),
				m.weekday_2(),
				m.weekday_3(),
				m.weekday_4(),
				m.weekday_5(),
				m.weekday_6()
			];
			if (cronWeekday.includes(',')) {
				const dayNums = cronWeekday.split(',');
				parts.push(dayNums.map((d) => days[parseInt(d)]).join('、'));
			} else {
				parts.push(days[parseInt(cronWeekday)]);
			}
		}

		if (parts.length === 0) return m.cron_execute_minute();
		return parts.join(' ');
	})();

	$: rruleDescription = (() => {
		const freqMap: Record<string, string> = {
			YEARLY: m.yearly(),
			MONTHLY: m.monthly(),
			WEEKLY: m.weekly(),
			DAILY: m.daily(),
			HOURLY: m.hourly()
		};

		const dayMap: Record<string, string> = {
			MO: m.weekday_1(),
			TU: m.weekday_2(),
			WE: m.weekday_3(),
			TH: m.weekday_4(),
			FR: m.weekday_5(),
			SA: m.weekday_6(),
			SU: m.weekday_0()
		};

		let desc = freqMap[rruleFreq] || rruleFreq;

		if (rruleInterval && rruleInterval !== '1') {
			desc = `${m.every()} ${rruleInterval} ${freqMap[rruleFreq].slice(1)}`;
		}

		if (rruleByDay.length > 0) {
			desc += ` (${rruleByDay.map((d) => dayMap[d]).join('、')})`;
		}

		if (rruleByMonthDay) {
			desc += ` ${m.on_specific_day({ day: rruleByMonthDay })}`;
		}

		if (rruleCount) {
			desc += ` ${m.total_times({ times: rruleCount })}`;
		}

		if (rruleUntil) {
			desc += ` ${m.until()} ${rruleUntil}`;
		}

		return desc;
	})();

	const copyToClipboard = (text: string) => {
		navigator.clipboard.writeText(text);
		copied = true;
		setTimeout(() => (copied = false), 2000);
	};

	const weekdays = [
		{ value: 'MO', label: m.weekday_1() },
		{ value: 'TU', label: m.weekday_2() },
		{ value: 'WE', label: m.weekday_3() },
		{ value: 'TH', label: m.weekday_4() },
		{ value: 'FR', label: m.weekday_5() },
		{ value: 'SA', label: m.weekday_6() },
		{ value: 'SU', label: m.weekday_0() }
	];

	const toggleWeekday = (day: string) => {
		if (rruleByDay.includes(day)) {
			rruleByDay = rruleByDay.filter((d) => d !== day);
		} else {
			rruleByDay = [...rruleByDay, day];
		}
	};

	const setCronPreset = (preset: string) => {
		switch (preset) {
			case 'midnight':
				cronMinute = '0';
				cronHour = '0';
				cronDay = '*';
				cronMonth = '*';
				cronWeekday = '*';
				break;
			case 'weekday9':
				cronMinute = '0';
				cronHour = '9';
				cronDay = '*';
				cronMonth = '*';
				cronWeekday = '1-5';
				break;
			case 'every5min':
				cronMinute = '*/5';
				cronHour = '*';
				cronDay = '*';
				cronMonth = '*';
				cronWeekday = '*';
				break;
			case 'every4hour':
				cronMinute = '0';
				cronHour = '*/4';
				cronDay = '*';
				cronMonth = '*';
				cronWeekday = '*';
				break;
		}
	};

	const handleLocaleChange = (e: Event) => {
		setLocale((e.target as HTMLSelectElement).value as 'en' | 'zh');
	};
</script>

<svelte:head>
	<style>
		@import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap');
	</style>
</svelte:head>

<div class="min-h-screen bg-linear-to-br from-blue-50 to-indigo-100 p-8">
	<div class="mx-auto max-w-4xl">
		<div class="rounded-2xl bg-white p-8 shadow-xl">
			<select name="locale" value={getLocale()} onchange={handleLocaleChange} class="mb-4 text-sm">
				<option value="zh">中文</option>
				<option value="en">EN</option>
			</select>
			<!-- Header -->
			<div class="mb-8 text-center">
				<h1 class="mb-2 flex items-center justify-center gap-3 text-4xl font-bold text-gray-800">
					<svg
						class="h-10 w-10 text-indigo-600"
						fill="none"
						stroke="currentColor"
						viewBox="0 0 24 24"
					>
						<path
							stroke-linecap="round"
							stroke-linejoin="round"
							stroke-width="2"
							d="M8 7V3m8 4V3m-9 8h10M5 21h14a2 2 0 002-2V7a2 2 0 00-2-2H5a2 2 0 00-2 2v12a2 2 0 002 2z"
						/>
					</svg>
					{m.title()}
				</h1>
				<p class="text-gray-600">{m.description()}</p>
			</div>

			<!-- Mode Selector -->
			<div class="mb-8 flex gap-4">
				<button
					onclick={() => (mode = 'cron')}
					class="flex-1 rounded-lg px-6 py-3 font-semibold transition-all {mode === 'cron'
						? 'bg-indigo-600 text-white shadow-lg'
						: 'bg-gray-100 text-gray-600 hover:bg-gray-200'}"
				>
					<svg class="mr-2 inline h-5 w-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
						<path
							stroke-linecap="round"
							stroke-linejoin="round"
							stroke-width="2"
							d="M12 8v4l3 3m6-3a9 9 0 11-18 0 9 9 0 0118 0z"
						/>
					</svg>
					{m.cron_expression()}
				</button>
				<button
					onclick={() => (mode = 'rrule')}
					class="flex-1 rounded-lg px-6 py-3 font-semibold transition-all {mode === 'rrule'
						? 'bg-indigo-600 text-white shadow-lg'
						: 'bg-gray-100 text-gray-600 hover:bg-gray-200'}"
				>
					<svg class="mr-2 inline h-5 w-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
						<path
							stroke-linecap="round"
							stroke-linejoin="round"
							stroke-width="2"
							d="M4 4v5h.582m15.356 2A8.001 8.001 0 004.582 9m0 0H9m11 11v-5h-.581m0 0a8.003 8.003 0 01-15.357-2m15.357 2H15"
						/>
					</svg>
					{m.rrule_expression()}
				</button>
			</div>

			<!-- Cron Editor -->
			{#if mode === 'cron'}
				<div class="space-y-6">
					<div class="grid grid-cols-1 gap-4 md:grid-cols-2">
						<div>
							<label class="mb-2 block text-sm font-semibold text-gray-700" for="cronMinute">
								{m.cron_minute()}
							</label>
							<input
								type="text"
								id="cronMinute"
								bind:value={cronMinute}
								placeholder="* 或 */5 或 0,15,30"
								class="w-full rounded-lg border-2 border-gray-200 px-4 py-2 focus:border-indigo-500 focus:outline-none"
							/>
							<p class="mt-1 text-xs text-gray-500">
								{m.cron_minute_desc()}
							</p>
						</div>

						<div>
							<label class="mb-2 block text-sm font-semibold text-gray-700" for="cronHour">
								{m.cron_hour()}
							</label>
							<input
								type="text"
								id="cronHour"
								bind:value={cronHour}
								placeholder="* 或 9 或 9-17"
								class="w-full rounded-lg border-2 border-gray-200 px-4 py-2 focus:border-indigo-500 focus:outline-none"
							/>
							<p class="mt-1 text-xs text-gray-500">{m.cron_hour_desc()}</p>
						</div>

						<div>
							<label class="mb-2 block text-sm font-semibold text-gray-700" for="cronDay">
								{m.cron_day()}
							</label>
							<input
								type="text"
								id="cronDay"
								bind:value={cronDay}
								placeholder="* 或 1 或 1,15"
								class="w-full rounded-lg border-2 border-gray-200 px-4 py-2 focus:border-indigo-500 focus:outline-none"
							/>
							<p class="mt-1 text-xs text-gray-500">{m.cron_day_desc()}</p>
						</div>

						<div>
							<label class="mb-2 block text-sm font-semibold text-gray-700" for="cronMonth">
								{m.cron_month()}
							</label>
							<input
								type="text"
								id="cronMonth"
								bind:value={cronMonth}
								placeholder="* 或 1 或 1,6,12"
								class="w-full rounded-lg border-2 border-gray-200 px-4 py-2 focus:border-indigo-500 focus:outline-none"
							/>
							<p class="mt-1 text-xs text-gray-500">{m.cron_month_desc()}</p>
						</div>

						<div class="md:col-span-2">
							<label class="mb-2 block text-sm font-semibold text-gray-700" for="cronWeekday">
								{m.cron_weekday()}
							</label>
							<input
								type="text"
								id="cronWeekday"
								bind:value={cronWeekday}
								placeholder="* 或 1 或 1-5"
								class="w-full rounded-lg border-2 border-gray-200 px-4 py-2 focus:border-indigo-500 focus:outline-none"
							/>
							<p class="mt-1 text-xs text-gray-500">
								{m.cron_weekday_desc()}
							</p>
						</div>
					</div>

					<!-- Quick presets -->
					<div>
						<label class="mb-2 block text-sm font-semibold text-gray-700" for="quickPresets">
							{m.cron_quick_select()}
						</label>
						<div class="flex flex-wrap gap-2">
							<button
								onclick={() => setCronPreset('midnight')}
								id="quickPresets"
								class="rounded-md bg-blue-100 px-3 py-1 text-sm text-blue-700 transition-colors hover:bg-blue-200"
							>
								{m.cron_quick_midnight()}
							</button>
							<button
								onclick={() => setCronPreset('weekday9')}
								class="rounded-md bg-blue-100 px-3 py-1 text-sm text-blue-700 transition-colors hover:bg-blue-200"
							>
								{m.cron_quick_weekday9()}
							</button>
							<button
								onclick={() => setCronPreset('every5min')}
								class="rounded-md bg-blue-100 px-3 py-1 text-sm text-blue-700 transition-colors hover:bg-blue-200"
							>
								{m.cron_quick_every5min()}
							</button>
							<button
								onclick={() => setCronPreset('every4hour')}
								class="rounded-md bg-blue-100 px-3 py-1 text-sm text-blue-700 transition-colors hover:bg-blue-200"
							>
								{m.cron_quick_every4hour()}
							</button>
						</div>
					</div>
				</div>
			{/if}

			<!-- RRule Editor -->
			{#if mode === 'rrule'}
				<div class="space-y-6">
					<div class="grid grid-cols-1 gap-4 md:grid-cols-2">
						<div>
							<label class="mb-2 block text-sm font-semibold text-gray-700" for="rruleFreq">
								{m.rr_frequency()}
							</label>
							<select
								id="rruleFreq"
								bind:value={rruleFreq}
								class="w-full rounded-lg border-2 border-gray-200 px-4 py-2 focus:border-indigo-500 focus:outline-none"
							>
								<option value="YEARLY">{m.rr_frequency_yearly()}</option>
								<option value="MONTHLY">{m.rr_frequency_monthly()}</option>
								<option value="WEEKLY">{m.rr_frequency_weekly()}</option>
								<option value="DAILY">{m.rr_frequency_daily()}</option>
								<option value="HOURLY">{m.rr_frequency_hourly()}</option>
							</select>
						</div>

						<div>
							<label class="mb-2 block text-sm font-semibold text-gray-700" for="rruleInterval">
								{m.rr_interval()}
							</label>
							<input
								id="rruleInterval"
								type="number"
								min="1"
								bind:value={rruleInterval}
								placeholder="1"
								class="w-full rounded-lg border-2 border-gray-200 px-4 py-2 focus:border-indigo-500 focus:outline-none"
							/>
							<p class="mt-1 text-xs text-gray-500">{m.rr_interval_desc()}</p>
						</div>

						<div>
							<label class="mb-2 block text-sm font-semibold text-gray-700" for="rruleCount">
								{m.rr_count()}
							</label>
							<input
								id="rruleCount"
								type="number"
								min="1"
								bind:value={rruleCount}
								placeholder={m.rr_count_placeholder()}
								class="w-full rounded-lg border-2 border-gray-200 px-4 py-2 focus:border-indigo-500 focus:outline-none"
							/>
						</div>

						<div>
							<label class="mb-2 block text-sm font-semibold text-gray-700" for="rruleUntil">
								{m.rr_until()}
							</label>
							<input
								id="rruleUntil"
								type="date"
								bind:value={rruleUntil}
								class="w-full rounded-lg border-2 border-gray-200 px-4 py-2 focus:border-indigo-500 focus:outline-none"
							/>
						</div>

						<div class="md:col-span-2">
							<label class="mb-2 block text-sm font-semibold text-gray-700" for="rruleByDay">
								{m.rr_byday()}
							</label>
							<div class="flex flex-wrap gap-2">
								{#each weekdays as day}
									<button
										id={`rruleByDay-${day.value}`}
										onclick={() => toggleWeekday(day.value)}
										class="rounded-lg px-4 py-2 font-medium transition-all {rruleByDay.includes(
											day.value
										)
											? 'bg-indigo-600 text-white'
											: 'bg-gray-100 text-gray-600 hover:bg-gray-200'}"
									>
										{day.label}
									</button>
								{/each}
							</div>
						</div>

						<div class="md:col-span-2">
							<label class="mb-2 block text-sm font-semibold text-gray-700" for="rruleByMonthDay">
								{m.rr_bymonthday()}
							</label>
							<input
								id="rruleByMonthDay"
								type="text"
								bind:value={rruleByMonthDay}
								placeholder={m.rr_bymonthday_placeholder()}
								class="w-full rounded-lg border-2 border-gray-200 px-4 py-2 focus:border-indigo-500 focus:outline-none"
							/>
						</div>
					</div>
				</div>
			{/if}

			<!-- Result Display -->
			<div class="mt-8 space-y-4">
				<div
					class="rounded-xl border-2 border-indigo-200 bg-linear-to-r from-indigo-50 to-blue-50 p-6"
				>
					<div class="mb-2 flex items-center justify-between">
						<h3 class="text-lg font-bold text-gray-800">
							{mode === 'cron' ? m.cron_expression() : m.rrule_expression()}
						</h3>
						<button
							onclick={() => copyToClipboard(mode === 'cron' ? cronExpression : rruleExpression)}
							class="flex items-center gap-2 rounded-lg bg-white px-3 py-1 transition-colors hover:bg-gray-50"
						>
							{#if copied}
								<svg
									class="h-4 w-4 text-green-600"
									fill="none"
									stroke="currentColor"
									viewBox="0 0 24 24"
								>
									<path
										stroke-linecap="round"
										stroke-linejoin="round"
										stroke-width="2"
										d="M5 13l4 4L19 7"
									/>
								</svg>
								<span class="text-sm text-green-600">{m.copied()}</span>
							{:else}
								<svg
									class="h-4 w-4 text-gray-600"
									fill="none"
									stroke="currentColor"
									viewBox="0 0 24 24"
								>
									<path
										stroke-linecap="round"
										stroke-linejoin="round"
										stroke-width="2"
										d="M8 16H6a2 2 0 01-2-2V6a2 2 0 012-2h8a2 2 0 012 2v2m-6 12h8a2 2 0 002-2v-8a2 2 0 00-2-2h-8a2 2 0 00-2 2v8a2 2 0 002 2z"
									/>
								</svg>
								<span class="text-sm text-gray-600">{m.copy()}</span>
							{/if}
						</button>
					</div>
					<code
						class="block rounded-lg bg-white px-4 py-3 font-mono text-lg break-all text-indigo-700"
					>
						{mode === 'cron' ? cronExpression : rruleExpression}
					</code>
				</div>

				<div
					class="rounded-xl border-2 border-green-200 bg-linear-to-r from-green-50 to-emerald-50 p-6"
				>
					<h3 class="mb-2 text-lg font-bold text-gray-800">{m.readable_description()}</h3>
					<p class="text-xl font-medium text-gray-700">
						{mode === 'cron' ? cronDescription : rruleDescription}
					</p>
				</div>
			</div>

			<!-- Info Box -->
			<div class="mt-8 rounded-xl border-2 border-amber-200 bg-amber-50 p-6">
				<h4 class="mb-2 font-bold text-amber-900">{m.usage_tips()}</h4>
				<ul class="space-y-1 text-sm text-amber-800">
					{#if mode === 'cron'}
						<li>• {m.usage_tips_cron()}</li>
						<li>• {m.usage_tips_cron_interval()}</li>
						<li>• {m.usage_tips_cron_multiple()}</li>
						<li>• {m.usage_tips_cron_range()}</li>
					{:else}
						<li>• {m.usage_tips_rrule()}</li>
						<li>• {m.usage_tips_rrule_interval()}</li>
						<li>• {m.usage_tips_rrule_count()}</li>
						<li>• {m.usage_tips_rrule_byday()}</li>
					{/if}
				</ul>
			</div>
		</div>
	</div>
</div>

<style>
	:global(body) {
		margin: 0;
		font-family:
			'Inter',
			-apple-system,
			BlinkMacSystemFont,
			'Segoe UI',
			'Roboto',
			'Oxygen',
			'Ubuntu',
			'Cantarell',
			'Fira Sans',
			'Droid Sans',
			'Helvetica Neue',
			sans-serif;
		-webkit-font-smoothing: antialiased;
		-moz-osx-font-smoothing: grayscale;
	}

	* {
		box-sizing: border-box;
	}
</style>
