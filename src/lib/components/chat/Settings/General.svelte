<script lang="ts">
	import { toast } from 'svelte-sonner';
	import { createEventDispatcher, onMount, getContext } from 'svelte';
	import { getLanguages, changeLanguage } from '$lib/i18n';
	const dispatch = createEventDispatcher();

	import { config, models, settings, theme, user } from '$lib/stores';

	const i18n = getContext('i18n');

	import AdvancedParams from './Advanced/AdvancedParams.svelte';
	import Textarea from '$lib/components/common/Textarea.svelte';
	export let saveSettings: Function;
	export let getModels: Function;

	// General
	let themes = ['dark', 'light', 'oled-dark', 'recube', 'recube-chiaro', 'recube-scuro'];
	let selectedTheme = 'system';
	// personalizzazione tema recube
	const recubePalette = ['#00243E', '#026172', '#3D97AD', '#EBB700', '#EC9400', '#FFFFFF'];
	let sidebarColor = '#00243E';
	let bodyColor = '#026172';
	let buttonColor = '#EBB700';
	const setRecubeColor = (type: string, color: string) => {
		if (type === 'sidebar') {
			sidebarColor = color;
			localStorage.setItem('recube-sidebar', color);
			if (selectedTheme === 'recube')
				document.documentElement.style.setProperty('--color-gray-950', color);
		}
		if (type === 'body') {
			bodyColor = color;
			localStorage.setItem('recube-body', color);
			if (selectedTheme === 'recube') {
				document.documentElement.style.setProperty('--color-gray-900', color);
				document.documentElement.style.setProperty('--color-gray-850', color);
			}
		}
		if (type === 'button') {
			buttonColor = color;
			localStorage.setItem('recube-button', color);
			if (selectedTheme === 'recube')
				document.documentElement.style.setProperty('--recube-accent', color);
		}
	};

	let languages: Awaited<ReturnType<typeof getLanguages>> = [];
	let lang = $i18n.language;
	let notificationEnabled = false;
	let system = '';

	let showAdvanced = false;

	const toggleNotification = async () => {
		const permission = await Notification.requestPermission();

		if (permission === 'granted') {
			notificationEnabled = !notificationEnabled;
			saveSettings({ notificationEnabled: notificationEnabled });
		} else {
			toast.error(
				$i18n.t(
					'Response notifications cannot be activated as the website permissions have been denied. Please visit your browser settings to grant the necessary access.'
				)
			);
		}
	};

	let params = {
		// Advanced
		stream_response: null,
		stream_delta_chunk_size: null,
		function_calling: null,
		seed: null,
		temperature: null,
		reasoning_effort: null,
		logit_bias: null,
		frequency_penalty: null,
		presence_penalty: null,
		repeat_penalty: null,
		repeat_last_n: null,
		mirostat: null,
		mirostat_eta: null,
		mirostat_tau: null,
		top_k: null,
		top_p: null,
		min_p: null,
		stop: null,
		tfs_z: null,
		num_ctx: null,
		num_batch: null,
		num_keep: null,
		max_tokens: null,
		num_gpu: null
	};

	const saveHandler = async () => {
		saveSettings({
			system: system !== '' ? system : undefined,
			params: {
				stream_response: params.stream_response !== null ? params.stream_response : undefined,
				stream_delta_chunk_size:
					params.stream_delta_chunk_size !== null ? params.stream_delta_chunk_size : undefined,
				function_calling: params.function_calling !== null ? params.function_calling : undefined,
				seed: (params.seed !== null ? params.seed : undefined) ?? undefined,
				stop: params.stop ? params.stop.split(',').filter((e) => e) : undefined,
				temperature: params.temperature !== null ? params.temperature : undefined,
				reasoning_effort: params.reasoning_effort !== null ? params.reasoning_effort : undefined,
				logit_bias: params.logit_bias !== null ? params.logit_bias : undefined,
				frequency_penalty: params.frequency_penalty !== null ? params.frequency_penalty : undefined,
				presence_penalty: params.frequency_penalty !== null ? params.frequency_penalty : undefined,
				repeat_penalty: params.frequency_penalty !== null ? params.frequency_penalty : undefined,
				repeat_last_n: params.repeat_last_n !== null ? params.repeat_last_n : undefined,
				mirostat: params.mirostat !== null ? params.mirostat : undefined,
				mirostat_eta: params.mirostat_eta !== null ? params.mirostat_eta : undefined,
				mirostat_tau: params.mirostat_tau !== null ? params.mirostat_tau : undefined,
				top_k: params.top_k !== null ? params.top_k : undefined,
				top_p: params.top_p !== null ? params.top_p : undefined,
				min_p: params.min_p !== null ? params.min_p : undefined,
				tfs_z: params.tfs_z !== null ? params.tfs_z : undefined,
				num_ctx: params.num_ctx !== null ? params.num_ctx : undefined,
				num_batch: params.num_batch !== null ? params.num_batch : undefined,
				num_keep: params.num_keep !== null ? params.num_keep : undefined,
				max_tokens: params.max_tokens !== null ? params.max_tokens : undefined,
				use_mmap: params.use_mmap !== null ? params.use_mmap : undefined,
				use_mlock: params.use_mlock !== null ? params.use_mlock : undefined,
				num_thread: params.num_thread !== null ? params.num_thread : undefined,
				num_gpu: params.num_gpu !== null ? params.num_gpu : undefined,
				think: params.think !== null ? params.think : undefined,
				keep_alive: params.keep_alive !== null ? params.keep_alive : undefined,
				format: params.format !== null ? params.format : undefined
			}
		});
		dispatch('save');
	};

	onMount(async () => {
		selectedTheme = localStorage.theme ?? 'system';
		sidebarColor = localStorage.getItem('recube-sidebar') || '#00243E';
		bodyColor = localStorage.getItem('recube-body') || '#026172';
		buttonColor = localStorage.getItem('recube-button') || '#EBB700';

		if (selectedTheme === 'recube') {
			document.documentElement.style.setProperty('--color-gray-950', sidebarColor);
			document.documentElement.style.setProperty('--color-gray-900', bodyColor);
			document.documentElement.style.setProperty('--color-gray-850', bodyColor);
			document.documentElement.style.setProperty('--recube-accent', buttonColor);
		}

		languages = await getLanguages();

		if (!$config?.features?.enable_easter_eggs) {
			languages = languages.filter((l) => l.code !== 'dg-DG');
		}

		notificationEnabled = $settings.notificationEnabled ?? false;
		system = $settings.system ?? '';

		params = { ...params, ...$settings.params };
		params.stop = $settings?.params?.stop ? ($settings?.params?.stop ?? []).join(',') : null;
	});

	const applyTheme = (_theme: string) => {
		let themeToApply =
			_theme === 'oled-dark'
				? 'dark'
				: _theme === 'her'
					? 'light'
					: _theme === 'recube' || _theme === 'recube-scuro'
						? 'dark'
						: _theme === 'recube-chiaro'
							? 'light'
							: _theme;

		if (_theme === 'system') {
			themeToApply = window.matchMedia('(prefers-color-scheme: dark)').matches ? 'dark' : 'light';
		}

		if (themeToApply === 'dark' && !_theme.includes('oled') && _theme !== 'recube') {
			document.documentElement.style.setProperty('--color-gray-800', '#333');
			document.documentElement.style.setProperty('--color-gray-850', '#262626');
			document.documentElement.style.setProperty('--color-gray-900', '#171717');
			document.documentElement.style.setProperty('--color-gray-950', '#0d0d0d');
		}

		themes
			.filter((e) => e !== themeToApply)
			.forEach((e) => {
				e.split(' ').forEach((e) => {
					document.documentElement.classList.remove(e);
				});
			});

		themeToApply.split(' ').forEach((e) => {
			document.documentElement.classList.add(e);
		});

		const metaThemeColor = document.querySelector('meta[name="theme-color"]');
		if (metaThemeColor) {
			if (_theme.includes('system')) {
				const systemTheme = window.matchMedia('(prefers-color-scheme: dark)').matches
					? 'dark'
					: 'light';
				console.log('Setting system meta theme color: ' + systemTheme);
				metaThemeColor.setAttribute('content', systemTheme === 'light' ? '#ffffff' : '#171717');
			} else {
				console.log('Setting meta theme color: ' + _theme);
				metaThemeColor.setAttribute(
					'content',
					_theme === 'dark' || _theme === 'recube-scuro'
						? '#171717'
						: _theme === 'oled-dark'
							? '#000000'
							: _theme === 'her'
								? '#983724'
								: _theme === 'recube'
									? '#00243E'
									: '#ffffff'
				);
			}
		}

		if (typeof window !== 'undefined' && window.applyTheme) {
			window.applyTheme();
		}

		if (_theme.includes('oled')) {
			document.documentElement.style.setProperty('--color-gray-800', '#101010');
			document.documentElement.style.setProperty('--color-gray-850', '#050505');
			document.documentElement.style.setProperty('--color-gray-900', '#000000');
			document.documentElement.style.setProperty('--color-gray-950', '#000000');
			document.documentElement.classList.add('dark');
		} else if (_theme === 'recube') {
			document.documentElement.style.setProperty('--color-gray-800', '#3D97AD');
			document.documentElement.style.setProperty(
				'--color-gray-850',
				localStorage.getItem('recube-body') || '#026172'
			);
			document.documentElement.style.setProperty(
				'--color-gray-900',
				localStorage.getItem('recube-body') || '#026172'
			);
			document.documentElement.style.setProperty(
				'--color-gray-950',
				localStorage.getItem('recube-sidebar') || '#00243E'
			);
			document.documentElement.style.setProperty(
				'--recube-accent',
				localStorage.getItem('recube-button') || '#EBB700'
			);
			document.documentElement.classList.add('dark');
			document.documentElement.classList.add('recube');
		} else if (_theme === 'recube-scuro') {
			document.documentElement.style.setProperty('--color-gray-900', '#000000');
			document.documentElement.style.setProperty('--color-gray-950', '#000000');
			document.documentElement.classList.add('dark');
			document.documentElement.classList.add('recube-scuro');
		} else if (_theme === 'recube-chiaro') {
			document.documentElement.classList.add('light');
			document.documentElement.classList.add('recube-chiaro');
		}

		console.log(_theme);
	};

	const themeChangeHandler = (_theme: string) => {
		theme.set(_theme);
		localStorage.setItem('theme', _theme);
		applyTheme(_theme);
	};
</script>

<div class="flex flex-col h-full justify-between text-sm" id="tab-general">
	<div class="  overflow-y-scroll max-h-[28rem] md:max-h-full">
		<div class="">
			<div class=" mb-1 text-sm font-medium">{$i18n.t('Settings')}</div>

			<div class="flex w-full justify-between">
				<div class=" self-center text-xs font-medium">{$i18n.t('Theme')}</div>
				<div class="flex items-center relative">
					<select
						class="w-fit pr-8 rounded-sm py-2 px-2 text-xs bg-transparent text-right {$settings.highContrastMode
							? ''
							: 'outline-hidden'}"
						bind:value={selectedTheme}
						placeholder={$i18n.t('Select a theme')}
						on:change={() => themeChangeHandler(selectedTheme)}
					>
						<option value="recube">Tema Personalizzato</option>
						<option value="recube-scuro">Recube Scuro</option>
						<option value="recube-chiaro">Recube Chiaro</option>
					</select>
				</div>
			</div>

			{#if selectedTheme === 'recube'}
				<div class="mt-4 space-y-4 border-t border-gray-100/30 dark:border-gray-850/30 pt-4 pb-2">
					<div class="text-sm font-medium">Personalizzazione Colori Recube</div>

					<div class="flex w-full justify-between items-center">
						<div class="text-xs font-medium">Colore barra laterale</div>
						<div class="flex gap-1.5 pr-2">
							{#each recubePalette.slice(0, 5) as color}
								<button
									type="button"
									class="w-5 h-5 rounded-full outline-hidden focus:outline-hidden border-2 transition-transform hover:scale-110 {sidebarColor ===
									color
										? 'border-gray-400 dark:border-white'
										: 'border-gray-400/50'}"
									style="background-color: {color};"
									on:click={() => setRecubeColor('sidebar', color)}
								></button>
							{/each}
						</div>
					</div>

					<div class="flex w-full justify-between items-center">
						<div class="text-xs font-medium">Colore sfondo</div>
						<div class="flex gap-1.5 pr-2">
							{#each recubePalette.slice(0, 2) as color}
								<button
									type="button"
									class="w-5 h-5 rounded-full outline-hidden focus:outline-hidden border-2 transition-transform hover:scale-110 {bodyColor ===
									color
										? 'border-gray-400 dark:border-white'
										: 'border-gray-400/50'}"
									style="background-color: {color};"
									on:click={() => setRecubeColor('body', color)}
								></button>
							{/each}
						</div>
					</div>

					<div class="flex w-full justify-between items-center">
						<div class="text-xs font-medium">Colore pulsanti</div>
						<div class="flex gap-1.5 pr-2">
							{#each recubePalette.slice(2) as color}
								<button
									type="button"
									class="w-5 h-5 rounded-full outline-hidden focus:outline-hidden border-2 transition-transform hover:scale-110 {buttonColor ===
									color
										? 'border-gray-400 dark:border-white'
										: 'border-gray-400/50'}"
									style="background-color: {color};"
									on:click={() => setRecubeColor('button', color)}
								></button>
							{/each}
						</div>
					</div>
				</div>
			{/if}
			<div class=" flex w-full justify-between">
				<div class=" self-center text-xs font-medium">{$i18n.t('Language')}</div>
				<div class="flex items-center relative">
					<select
						class="w-fit pr-8 rounded-sm py-2 px-2 text-xs bg-transparent text-right {$settings.highContrastMode
							? ''
							: 'outline-hidden'}"
						bind:value={lang}
						placeholder={$i18n.t('Select a language')}
						on:change={(e) => {
							changeLanguage(lang);
						}}
					>
						{#each languages as language}
							<option value={language['code']}>{language['title']}</option>
						{/each}
					</select>
				</div>
			</div>
			{#if $i18n.language === 'en-US' && !($config?.license_metadata ?? false)}
				<div
					class="mb-2 text-xs {($settings?.highContrastMode ?? false)
						? 'text-gray-800 dark:text-gray-100'
						: 'text-gray-400 dark:text-gray-500'}"
				>
					Couldn't find your language?
					<a
						class="font-medium underline {($settings?.highContrastMode ?? false)
							? 'text-gray-700 dark:text-gray-200'
							: 'text-gray-300'}"
						href="https://github.com/open-webui/open-webui/blob/main/docs/CONTRIBUTING.md#-translations-and-internationalization"
						target="_blank"
					>
						Help us translate Open WebUI!
					</a>
				</div>
			{/if}

			{#if $user?.role === 'admin'}
				<div>
					<div class=" py-0.5 flex w-full justify-between">
						<div class=" self-center text-xs font-medium">{$i18n.t('Notifications')}</div>

						<button
							class="p-1 px-3 text-xs flex rounded-sm transition"
							on:click={() => {
								toggleNotification();
							}}
							type="button"
							role="switch"
							aria-checked={notificationEnabled}
						>
							{#if notificationEnabled === true}
								<span class="ml-2 self-center">{$i18n.t('On')}</span>
							{:else}
								<span class="ml-2 self-center">{$i18n.t('Off')}</span>
							{/if}
						</button>
					</div>
				</div>
			{/if}
		</div>

		{#if $user?.role === 'admin'}
			<hr class="border-gray-100/30 dark:border-gray-850/30 my-3" />

			<div>
				<div class=" my-2.5 text-sm font-medium">{$i18n.t('System Prompt')}</div>
				<Textarea
					bind:value={system}
					className={'w-full text-sm outline-hidden ' +
						($settings.highContrastMode
							? ' p-2.5 border-2 border-gray-300 dark:border-gray-700 rounded-lg bg-transparent text-gray-900 dark:text-gray-100 focus:ring-1 focus:ring-blue-500 focus:border-blue-500 overflow-y-hidden'
							: '  dark:text-gray-300 ')}
					rows="4"
					placeholder={$i18n.t('Enter system prompt here')}
				/>
			</div>
		{/if}

		{#if $user?.role === 'admin'}
			<div class="mt-2 space-y-3 pr-1.5">
				<div class="flex justify-between items-center text-sm">
					<div class="  font-medium">{$i18n.t('Advanced Parameters')}</div>
					<button
						class=" text-xs font-medium {($settings?.highContrastMode ?? false)
							? 'text-gray-800 dark:text-gray-100'
							: 'text-gray-400 dark:text-gray-500'}"
						type="button"
						aria-expanded={showAdvanced}
						on:click={() => {
							showAdvanced = !showAdvanced;
						}}>{showAdvanced ? $i18n.t('Hide') : $i18n.t('Show')}</button
					>
				</div>

				{#if showAdvanced}
					<AdvancedParams admin={$user?.role === 'admin'} bind:params />
				{/if}
			</div>
		{/if}
	</div>

	<div class="flex justify-end pt-3 text-sm font-medium">
		<button
			class="px-3.5 py-1.5 text-sm font-medium bg-black hover:bg-gray-900 text-white dark:bg-white dark:text-black dark:hover:bg-gray-100 recube:bg-[var(--recube-accent)] recube:text-black recube:hover:opacity-80 transition rounded-full"
			on:click={() => {
				saveHandler();
			}}
		>
			{$i18n.t('Save')}
		</button>
	</div>
</div>
