<script lang="ts">
	import { onMount } from 'svelte';

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
			return '每分钟执行';
		}

		if (cronMinute !== '*') {
			if (cronMinute.includes('/')) {
				const interval = cronMinute.split('/')[1];
				parts.push(`每${interval}分钟`);
			} else if (cronMinute.includes(',')) {
				parts.push(`在第 ${cronMinute} 分钟`);
			} else {
				parts.push(`在第 ${cronMinute} 分钟`);
			}
		}

		if (cronHour !== '*') {
			if (cronHour.includes('/')) {
				const interval = cronHour.split('/')[1];
				parts.push(`每${interval}小时`);
			} else {
				parts.push(`${cronHour}点`);
			}
		}

		if (cronDay !== '*') {
			parts.push(`每月第 ${cronDay} 天`);
		}

		if (cronMonth !== '*') {
			const months = [
				'1月',
				'2月',
				'3月',
				'4月',
				'5月',
				'6月',
				'7月',
				'8月',
				'9月',
				'10月',
				'11月',
				'12月'
			];
			if (cronMonth.includes(',')) {
				const monthNums = cronMonth.split(',');
				parts.push(monthNums.map((m) => months[parseInt(m) - 1]).join('、'));
			} else {
				parts.push(months[parseInt(cronMonth) - 1]);
			}
		}

		if (cronWeekday !== '*') {
			const days = ['周日', '周一', '周二', '周三', '周四', '周五', '周六'];
			if (cronWeekday.includes(',')) {
				const dayNums = cronWeekday.split(',');
				parts.push(dayNums.map((d) => days[parseInt(d)]).join('、'));
			} else {
				parts.push(days[parseInt(cronWeekday)]);
			}
		}

		if (parts.length === 0) return '每分钟执行';
		return parts.join(' ');
	})();

	$: rruleDescription = (() => {
		const freqMap: Record<string, string> = {
			YEARLY: '每年',
			MONTHLY: '每月',
			WEEKLY: '每周',
			DAILY: '每天',
			HOURLY: '每小时'
		};

		const dayMap: Record<string, string> = {
			MO: '周一',
			TU: '周二',
			WE: '周三',
			TH: '周四',
			FR: '周五',
			SA: '周六',
			SU: '周日'
		};

		let desc = freqMap[rruleFreq] || rruleFreq;

		if (rruleInterval && rruleInterval !== '1') {
			desc = `每 ${rruleInterval} ${freqMap[rruleFreq].slice(1)}`;
		}

		if (rruleByDay.length > 0) {
			desc += ` (${rruleByDay.map((d) => dayMap[d]).join('、')})`;
		}

		if (rruleByMonthDay) {
			desc += ` 第 ${rruleByMonthDay} 天`;
		}

		if (rruleCount) {
			desc += ` 共 ${rruleCount} 次`;
		}

		if (rruleUntil) {
			desc += ` 直到 ${rruleUntil}`;
		}

		return desc;
	})();

	const copyToClipboard = (text: string) => {
		navigator.clipboard.writeText(text);
		copied = true;
		setTimeout(() => (copied = false), 2000);
	};

	const weekdays = [
		{ value: 'MO', label: '周一' },
		{ value: 'TU', label: '周二' },
		{ value: 'WE', label: '周三' },
		{ value: 'TH', label: '周四' },
		{ value: 'FR', label: '周五' },
		{ value: 'SA', label: '周六' },
		{ value: 'SU', label: '周日' }
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
</script>

<svelte:head>
	<style>
		@import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap');
	</style>
</svelte:head>

<div class="min-h-screen bg-gradient-to-br from-blue-50 to-indigo-100 p-8">
	<div class="mx-auto max-w-4xl">
		<div class="rounded-2xl bg-white p-8 shadow-xl">
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
					定时任务可视化编辑器
				</h1>
				<p class="text-gray-600">轻松创建和管理 Cron 和 RRule 表达式</p>
			</div>

			<!-- Mode Selector -->
			<div class="mb-8 flex gap-4">
				<button
					on:click={() => (mode = 'cron')}
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
					Cron 表达式
				</button>
				<button
					on:click={() => (mode = 'rrule')}
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
					RRule 规则
				</button>
			</div>

			<!-- Cron Editor -->
			{#if mode === 'cron'}
				<div class="space-y-6">
					<div class="grid grid-cols-1 gap-4 md:grid-cols-2">
						<div>
							<label class="mb-2 block text-sm font-semibold text-gray-700" for="cronMinute">
								分钟 (0-59)
							</label>
							<input
								type="text"
								id="cronMinute"
								bind:value={cronMinute}
								placeholder="* 或 */5 或 0,15,30"
								class="w-full rounded-lg border-2 border-gray-200 px-4 py-2 focus:border-indigo-500 focus:outline-none"
							/>
							<p class="mt-1 text-xs text-gray-500">
								* = 每分钟, */5 = 每5分钟, 0,30 = 第0和30分钟
							</p>
						</div>

						<div>
							<label class="mb-2 block text-sm font-semibold text-gray-700" for="cronHour">
								小时 (0-23)
							</label>
							<input
								type="text"
								id="cronHour"
								bind:value={cronHour}
								placeholder="* 或 9 或 9-17"
								class="w-full rounded-lg border-2 border-gray-200 px-4 py-2 focus:border-indigo-500 focus:outline-none"
							/>
							<p class="mt-1 text-xs text-gray-500">* = 每小时, 9 = 9点, 9-17 = 9点到17点</p>
						</div>

						<div>
							<label class="mb-2 block text-sm font-semibold text-gray-700" for="cronDay">
								日期 (1-31)
							</label>
							<input
								type="text"
								id="cronDay"
								bind:value={cronDay}
								placeholder="* 或 1 或 1,15"
								class="w-full rounded-lg border-2 border-gray-200 px-4 py-2 focus:border-indigo-500 focus:outline-none"
							/>
							<p class="mt-1 text-xs text-gray-500">* = 每天, 1 = 每月1号, 1,15 = 1号和15号</p>
						</div>

						<div>
							<label class="mb-2 block text-sm font-semibold text-gray-700" for="cronMonth">
								月份 (1-12)
							</label>
							<input
								type="text"
								id="cronMonth"
								bind:value={cronMonth}
								placeholder="* 或 1 或 1,6,12"
								class="w-full rounded-lg border-2 border-gray-200 px-4 py-2 focus:border-indigo-500 focus:outline-none"
							/>
							<p class="mt-1 text-xs text-gray-500">* = 每月, 1 = 1月, 1,6,12 = 1、6、12月</p>
						</div>

						<div class="md:col-span-2">
							<label class="mb-2 block text-sm font-semibold text-gray-700" for="cronWeekday">
								星期 (0-6, 0=周日)
							</label>
							<input
								type="text"
								id="cronWeekday"
								bind:value={cronWeekday}
								placeholder="* 或 1 或 1-5"
								class="w-full rounded-lg border-2 border-gray-200 px-4 py-2 focus:border-indigo-500 focus:outline-none"
							/>
							<p class="mt-1 text-xs text-gray-500">
								* = 每天, 1 = 周一, 1-5 = 周一到周五, 0,6 = 周末
							</p>
						</div>
					</div>

					<!-- Quick presets -->
					<div>
						<label class="mb-2 block text-sm font-semibold text-gray-700" for="quickPresets">
							快速选择
						</label>
						<div class="flex flex-wrap gap-2">
							<button
								on:click={() => setCronPreset('midnight')}
								id="quickPresets"
								class="rounded-md bg-blue-100 px-3 py-1 text-sm text-blue-700 transition-colors hover:bg-blue-200"
							>
								每天午夜
							</button>
							<button
								on:click={() => setCronPreset('weekday9')}
								class="rounded-md bg-blue-100 px-3 py-1 text-sm text-blue-700 transition-colors hover:bg-blue-200"
							>
								工作日早9点
							</button>
							<button
								on:click={() => setCronPreset('every5min')}
								class="rounded-md bg-blue-100 px-3 py-1 text-sm text-blue-700 transition-colors hover:bg-blue-200"
							>
								每5分钟
							</button>
							<button
								on:click={() => setCronPreset('every4hour')}
								class="rounded-md bg-blue-100 px-3 py-1 text-sm text-blue-700 transition-colors hover:bg-blue-200"
							>
								每4小时
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
								频率 (FREQ)
							</label>
							<select
								id="rruleFreq"
								bind:value={rruleFreq}
								class="w-full rounded-lg border-2 border-gray-200 px-4 py-2 focus:border-indigo-500 focus:outline-none"
							>
								<option value="YEARLY">每年</option>
								<option value="MONTHLY">每月</option>
								<option value="WEEKLY">每周</option>
								<option value="DAILY">每天</option>
								<option value="HOURLY">每小时</option>
							</select>
						</div>

						<div>
							<label class="mb-2 block text-sm font-semibold text-gray-700" for="rruleInterval">
								间隔 (INTERVAL)
							</label>
							<input
								id="rruleInterval"
								type="number"
								min="1"
								bind:value={rruleInterval}
								placeholder="1"
								class="w-full rounded-lg border-2 border-gray-200 px-4 py-2 focus:border-indigo-500 focus:outline-none"
							/>
							<p class="mt-1 text-xs text-gray-500">例如: 2 = 每隔一次</p>
						</div>

						<div>
							<label class="mb-2 block text-sm font-semibold text-gray-700" for="rruleCount">
								重复次数 (COUNT)
							</label>
							<input
								id="rruleCount"
								type="number"
								min="1"
								bind:value={rruleCount}
								placeholder="留空表示无限重复"
								class="w-full rounded-lg border-2 border-gray-200 px-4 py-2 focus:border-indigo-500 focus:outline-none"
							/>
						</div>

						<div>
							<label class="mb-2 block text-sm font-semibold text-gray-700" for="rruleUntil">
								结束日期 (UNTIL)
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
								指定星期几 (BYDAY)
							</label>
							<div class="flex flex-wrap gap-2">
								{#each weekdays as day}
									<button
										id={`rruleByDay-${day.value}`}
										on:click={() => toggleWeekday(day.value)}
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
								指定日期 (BYMONTHDAY)
							</label>
							<input
								id="rruleByMonthDay"
								type="text"
								bind:value={rruleByMonthDay}
								placeholder="例如: 1 或 1,15 或 -1 (最后一天)"
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
							{mode === 'cron' ? 'Cron 表达式' : 'RRule 规则'}
						</h3>
						<button
							on:click={() => copyToClipboard(mode === 'cron' ? cronExpression : rruleExpression)}
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
								<span class="text-sm text-green-600">已复制</span>
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
								<span class="text-sm text-gray-600">复制</span>
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
					<h3 class="mb-2 text-lg font-bold text-gray-800">可读描述</h3>
					<p class="text-xl font-medium text-gray-700">
						{mode === 'cron' ? cronDescription : rruleDescription}
					</p>
				</div>
			</div>

			<!-- Info Box -->
			<div class="mt-8 rounded-xl border-2 border-amber-200 bg-amber-50 p-6">
				<h4 class="mb-2 font-bold text-amber-900">💡 使用提示</h4>
				<ul class="space-y-1 text-sm text-amber-800">
					{#if mode === 'cron'}
						<li>• 使用 * 表示"每个"时间单位</li>
						<li>• 使用 */n 表示"每隔 n 个"时间单位</li>
						<li>• 使用 1,2,3 表示多个特定值</li>
						<li>• 使用 1-5 表示范围</li>
					{:else}
						<li>• FREQ 决定重复的基本频率</li>
						<li>• INTERVAL 设置间隔倍数</li>
						<li>• COUNT 和 UNTIL 不能同时使用</li>
						<li>• BYDAY 在不同 FREQ 下有不同含义</li>
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
