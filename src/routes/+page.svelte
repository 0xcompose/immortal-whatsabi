<script lang="ts">
	import { createPublicClient, http, isAddress } from 'viem'
	import * as chains from 'viem/chains'
	import { whatsabi, loaders } from '@shazow/whatsabi'

	console.log('chains', chains.mainnet)

	interface SelectedChain {
		name: string
		chain: chains.Chain
	}

	// Click outside action
	function clickOutside(node: HTMLElement, handler: () => void) {
		const handleClick = (event: MouseEvent) => {
			if (!node.contains(event.target as Node)) {
				handler()
			}
		}

		document.addEventListener('click', handleClick, true)

		return {
			destroy() {
				document.removeEventListener('click', handleClick, true)
			}
		}
	}

	function splitWordsFromCamelCase(str: string) {
		return str
			.replace(/([A-Z])/g, ' $1') // Add spaces before capital letters
			.trim()
			.split(' ')
			.map((word: string) => word.charAt(0).toUpperCase() + word.slice(1)) // Capitalize each word
			.join(' ')
	}

	// Convert chains object to array and filter out non-chain exports
	const allChains: SelectedChain[] = Object.entries(chains)
		.filter(([_, value]) => value && typeof value === 'object' && 'id' in value)
		.map(([key, chain]) => ({
			name: splitWordsFromCamelCase(key),
			chain
		}))
		.sort((a, b) => a.name.localeCompare(b.name))

	const mainnetIndex = allChains.findIndex((chain) => {
		if (chain.chain.id === 1) {
			return chain
		}
	})

	allChains[mainnetIndex].name = 'Ethereum Mainnet'
	const temp = allChains[0]
	allChains[0] = allChains[mainnetIndex]
	allChains[mainnetIndex] = temp

	let inputValue = ''
	let inputError = ''

	let chainSearch = ''
	let selectedChain: SelectedChain | undefined
	let showChainDropdown = false

	let copySuccess = false
	let resolvedABI: any[] = []
	let contractResult: any = null

	let notice = ''
	let error = ''

	let isLoading = false
	let currentPhase = ''
	
	let etherscanApiKey = ''
	let apiKeySaved = false

	// Load API key from localStorage on mount
	if (typeof window !== 'undefined') {
		const savedKey = localStorage.getItem('etherscanApiKey')
		if (savedKey) {
			etherscanApiKey = savedKey
		}
	}

	function saveApiKey() {
		if (etherscanApiKey.trim()) {
			localStorage.setItem('etherscanApiKey', etherscanApiKey)
		} else {
			localStorage.removeItem('etherscanApiKey')
		}
		apiKeySaved = true
		setTimeout(() => (apiKeySaved = false), 2000)
	}

	$: filteredChains = chainSearch
		? allChains.filter((chain) => chain.name.toLowerCase().includes(chainSearch.toLowerCase()))
		: allChains

	function selectChain(chain: SelectedChain) {
		selectedChain = chain
		chainSearch = chain.name
		showChainDropdown = false
	}

	function closeDropdown() {
		showChainDropdown = false
	}

	async function validateAddress() {
		if (!inputValue) {
			inputError = 'Address is required'
			return false
		}

		if (inputValue.endsWith('.eth') || isAddress(inputValue)) {
			inputError = ''
			return true
		}

		inputError = 'Invalid Ethereum address or ENS name'
		return false
	}

	async function getABI() {
		isLoading = true
		error = ''
		notice = ''
		resolvedABI = []
		contractResult = null

		if (!selectedChain) {
			error = 'No chain selected'
			isLoading = false
			return
		}

		try {
			console.log('selectedChain', selectedChain)
			const client = createPublicClient({
				chain: selectedChain.chain,
				transport: http()
			})

			// Configure ABI loader with Etherscan API key if provided
			const config: any = {
				provider: client,
				followProxies: true,
				loadContractResult: true,
				onProgress: (phase: string) => {
					currentPhase = splitWordsFromCamelCase(phase) + '...'
				}
			}

			// Add Etherscan loader if API key is provided
			if (etherscanApiKey.trim()) {
				try {
					config.abiLoader = new loaders.MultiABILoader([
						new loaders.SourcifyABILoader({ chainId: selectedChain.chain.id }),
						new loaders.EtherscanV2ABILoader({
							apiKey: etherscanApiKey,
							chainId: selectedChain.chain.id
						})
					])
				} catch (e) {
					console.warn('Failed to initialize Etherscan loader:', e)
				}
			}

			const {
				abi,
				address: resolvedAddress,
				hasCode,
				contractResult: result
			} = await whatsabi.autoload(inputValue, config)

			if (!hasCode) {
				error = 'This address is not a contract'
				isLoading = false
				return
			}

			resolvedABI = abi
			contractResult = result

			if (resolvedAddress.toLowerCase() !== inputValue.toLowerCase()) {
				notice = `Note: This is a proxy contract. The actual implementation address is ${resolvedAddress}`
			} else {
				notice = ''
			}
			console.log('Implementation Address', resolvedAddress)
		} catch (err: any) {
			console.error('Error fetching ABI:', err)
			error = err instanceof Error ? err.message : 'Failed to fetch ABI'
		} finally {
			isLoading = false
		}
	}

	async function copyABI() {
		try {
			await navigator.clipboard.writeText(JSON.stringify(resolvedABI, null, 2))
			copySuccess = true
			setTimeout(() => (copySuccess = false), 2000)
		} catch (error) {
			console.error('Error copying to clipboard:', error)
		}
	}

	function handleInput() {
		if (inputValue) validateAddress()
		else inputError = ''
	}
</script>

<svelte:head>
	<title>Immortal WhatsABI - Extract ABI from any EVM Contract</title>
	<meta name="description" content="Extract contract ABIs from any EVM address using WhatsABI, Sourcify, and Etherscan. Works with verified and unverified contracts across all EVM chains." />
</svelte:head>

<section>
	<h1>🔍💀 Immortal WhatsABI</h1>
	<div class="header-info">
		<a
			href="https://github.com/0xcompose"
			target="_blank"
			rel="noopener noreferrer"
			class="author-link"
		>
			Made with 💚 by 0xcompose
		</a>
		<div class="tools">
			<a
				href="https://github.com/wagmi-dev/viem"
				target="_blank"
				rel="noopener noreferrer"
				class="tool-tag"
			>
				Viem v2.38.6
			</a>
			<a
				href="https://github.com/shazow/whatsabi"
				target="_blank"
				rel="noopener noreferrer"
				class="tool-tag"
			>
				WhatsABI v0.23.0
			</a>
			<a href="https://hosting.fleek.xyz" target="_blank" rel="noopener noreferrer" class="tool-tag">
				Fleek (IPFS)
			</a>
			<a
				href="https://github.com/0xcompose/immortal-whatsabi"
				target="_blank"
				rel="noopener noreferrer"
				class="tool-tag"
			>
				GitHub
			</a>
			<a href="https://svelte.dev" target="_blank" rel="noopener noreferrer" class="tool-tag">
				Svelte v5.43.4
			</a>
		</div>
	</div>
	<div class="input-container">
		<div class="main-input-row">
			<div class="input-group">
				<div class="input-wrapper">
					<input
						type="text"
						bind:value={inputValue}
						on:input={handleInput}
						placeholder="Enter contract address or ENS name"
						class:error={inputError}
					/>
					{#if inputError}
						<span class="error-message">{inputError}</span>
					{/if}
				</div>
				<div class="chain-search-wrapper" use:clickOutside={closeDropdown}>
					<div class="input-with-icon">
						<input
							type="text"
							bind:value={chainSearch}
							on:focus={() => (showChainDropdown = true)}
							placeholder="Search chain..."
							class="chain-search"
						/>
						<svg
							class="search-icon"
							viewBox="0 0 24 24"
							fill="none"
							xmlns="http://www.w3.org/2000/svg"
						>
							<path
								d="M15.5 14H14.71L14.43 13.73C15.41 12.59 16 11.11 16 9.5C16 5.91 13.09 3 9.5 3C5.91 3 3 5.91 3 9.5C3 13.09 5.91 16 9.5 16C11.11 16 12.59 15.41 13.73 14.43L14 14.71V15.5L19 20.49L20.49 19L15.5 14ZM9.5 14C7.01 14 5 11.99 5 9.5C5 7.01 7.01 5 9.5 5C11.99 5 14 7.01 14 9.5C14 11.99 11.99 14 9.5 14Z"
								fill="currentColor"
							/>
						</svg>
					</div>
					{#if showChainDropdown && filteredChains.length > 0}
						<div class="chain-dropdown">
							{#each filteredChains as chain}
								<button
									class="chain-option"
									class:selected={chain === selectedChain}
									on:click={() => selectChain(chain)}
								>
									{chain.name}
								</button>
							{/each}
						</div>
					{/if}
				</div>
			</div>
			<button on:click={getABI} disabled={isLoading}> Get ABI </button>
		</div>
		
		<div class="api-key-row">
			<input
				id="etherscan-api-key"
				type="text"
				bind:value={etherscanApiKey}
				placeholder="Etherscan API Key (optional - improves ABI detection)"
			/>
			<button class="save-button" on:click={saveApiKey}>
				{apiKeySaved ? '✓ Saved' : 'Save'}
			</button>
		</div>
	</div>

	{#if isLoading}
		<div class="loader-container">
			<div class="loader">
				<div class="loader-spinner"></div>
				<span class="loader-text">{currentPhase || 'Loading...'}</span>
			</div>
		</div>
	{/if}

	{#if error?.length > 0}
		<div class="error-container">
			<p class="error-message">{error}</p>
		</div>
	{/if}

	{#if notice}
		<div class="info-container">
			<p class="info-message">{notice}</p>
		</div>
	{/if}

	{#if contractResult && contractResult.ok}
		<div class="contract-info">
			<h3>Contract Information</h3>
			<div class="info-grid">
				{#if contractResult.name}
					<div class="info-item">
						<span class="info-label">Name:</span>
						<span class="info-value">{contractResult.name}</span>
					</div>
				{/if}
				{#if contractResult.compilerVersion}
					<div class="info-item">
						<span class="info-label">Compiler:</span>
						<span class="info-value">{contractResult.compilerVersion}</span>
					</div>
				{/if}
				{#if contractResult.evmVersion}
					<div class="info-item">
						<span class="info-label">EVM Version:</span>
						<span class="info-value">{contractResult.evmVersion}</span>
					</div>
				{/if}
				{#if contractResult.runs !== undefined}
					<div class="info-item">
						<span class="info-label">Optimization Runs:</span>
						<span class="info-value">{contractResult.runs}</span>
					</div>
				{/if}
				{#if contractResult.loader?.name}
					<div class="info-item">
						<span class="info-label">Source:</span>
						<span class="info-value">{contractResult.loader.name.replace('ABILoader', '')}</span>
					</div>
				{/if}
			</div>
		</div>
	{/if}

	{#if resolvedABI.length > 0}
		<div class="output-container">
			<div class="output-header">
				<h2>Contract ABI</h2>
				<button class="copy-button" on:click={copyABI}>
					{copySuccess ? 'Copied!' : 'Copy'}
				</button>
			</div>
			<pre>{JSON.stringify(resolvedABI, null, 2)}</pre>
		</div>
	{/if}

	<footer></footer>
</section>

<style>
	section {
		display: flex;
		flex-direction: column;
		justify-content: flex-start;
		align-items: center;
		flex: 1;
		gap: 1rem;
		padding: 1rem;
		max-width: 1200px;
		margin: 0 auto;
		width: 100%;
		box-sizing: border-box;
	}

	/* Tablet styles */
	@media (min-width: 600px) and (max-width: 1023px) {
		section {
			padding: 1.5rem;
			gap: 1.5rem;
		}

		h1 {
			font-size: 2.25rem;
		}

		.input-container {
			max-width: 600px;
		}

		.input-group {
			flex-direction: row;
			gap: 1rem;
		}

		.chain-search-wrapper {
			width: 180px;
			min-width: 180px;
		}

		pre {
			font-size: 0.875rem;
			padding: 1.25rem;
		}

		.output-container {
			max-width: 600px;
		}

		.output-header {
			flex-direction: row;
			justify-content: space-between;
			align-items: center;
			padding: 1rem 1.25rem;
			gap: 1rem;
		}

		.output-header h2 {
			margin: 0;
		}
	}

	/* Desktop styles */
	@media (min-width: 1024px) {
		section {
			padding: 2rem;
		}

		h1 {
			font-size: 2.5rem;
		}

		.input-container {
			max-width: 800px;
		}

		.main-input-row {
			flex-direction: row;
		}

		.input-group {
			flex-direction: row;
		}

		.chain-search-wrapper {
			width: 200px;
		}

		.output-container {
			max-width: 800px;
		}

		pre {
			padding: 1.5rem;
			font-size: 0.9rem;
			white-space: pre;
			word-break: normal;
		}

		.output-header {
			flex-direction: row;
			justify-content: space-between;
			align-items: center;
			padding: 1rem 1.5rem;
		}
	}

	h1 {
		margin-bottom: 0.25rem;
		color: #1b5e20;
		font-size: 2rem;
		font-weight: 600;
		text-align: center;
	}

	h2 {
		color: #1b5e20;
		font-weight: 500;
	}

	.input-container {
		display: flex;
		flex-direction: column;
		gap: 1rem;
		width: 100%;
		max-width: 800px;
	}

	.main-input-row {
		display: flex;
		flex-direction: row;
		gap: 1rem;
		align-items: flex-start;
	}

	.input-group {
		display: flex;
		flex-direction: row;
		gap: 0.75rem;
		flex: 1;
	}

	.input-wrapper {
		display: flex;
		flex-direction: column;
		gap: 0.25rem;
		flex: 1;
	}

	.api-key-row {
		display: flex;
		gap: 0.5rem;
		align-items: center;
		width: 100%;
	}

	.api-key-row input {
		flex: 1;
	}

	.save-button {
		padding: 0.75rem 1.5rem;
		border: none;
		border-radius: 4px;
		font-weight: 500;
		cursor: pointer;
		transition: all 0.2s ease;
		white-space: nowrap;
		min-width: fit-content;
		background-color: #2e7d32;
		color: white;
		display: inline-flex;
		align-items: center;
		justify-content: center;
	}

	.save-button:hover:not(:disabled) {
		background-color: #1b5e20;
	}

	.save-button:disabled {
		background-color: #ccc;
		cursor: not-allowed;
	}

	.chain-search-wrapper {
		position: relative;
		width: 180px;
		min-width: 180px;
	}

	.chain-dropdown {
		position: absolute;
		top: 100%;
		left: 0;
		right: 0;
		max-height: 300px;
		overflow-y: auto;
		background: white;
		border: 1px solid #e0e0e0;
		border-radius: 4px;
		box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
		z-index: 10;
	}

	.chain-option {
		width: 100%;
		padding: 0.75rem;
		text-align: left;
		background: none;
		border: none;
		cursor: pointer;
		transition: all 0.2s ease;
		color: #333;
		font-size: 0.9rem;
	}

	.chain-option:hover,
	.chain-option:focus {
		background-color: #2e7d32;
		color: white;
		outline: none;
	}

	.chain-option.selected {
		background-color: #2e7d32;
		color: white;
		font-weight: 500;
	}

	input,
	.chain-search {
		padding: 0.75rem;
		border: 1px solid #e0e0e0;
		border-radius: 6px;
		font-size: 1rem;
		transition: all 0.2s ease;
		background: #f8f9fa;
		height: 42px;
		box-sizing: border-box;
		width: 100%;
	}

	@media (min-width: 600px) and (max-width: 1023px) {
		input,
		.chain-search {
			font-size: 0.95rem;
			padding: 0.7rem;
		}

		button {
			padding: 0 1.25rem;
			min-width: 90px;
		}
	}

	input:focus,
	.chain-search:focus {
		outline: none;
		border-color: #2e7d32;
		background: white;
		box-shadow: 0 0 0 3px rgba(46, 125, 50, 0.1);
	}

	input.error {
		border-color: #d32f2f;
		background: #fff5f5;
	}

	.error-message {
		color: #d32f2f;
		font-size: 0.875rem;
	}

	.error-container {
		padding: 1rem 1.5rem;
		background: #fff5f5;
		border: 1px solid #ffcdd2;
		border-radius: 6px;
		width: 100%;
		max-width: 800px;
		color: #d32f2f;
		display: flex;
		align-items: center;
		gap: 0.5rem;
	}

	button {
		padding: 0 1.5rem;
		background: #2e7d32;
		color: white;
		border: none;
		border-radius: 6px;
		cursor: pointer;
		height: 42px;
		font-weight: 500;
		letter-spacing: 0.5px;
		transition: all 0.2s ease;
		min-width: 100px;
	}

	button:disabled {
		background: #66bb6a;
		cursor: not-allowed;
		opacity: 0.8;
		color: rgba(255, 255, 255, 0.9);
	}

	button:not(:disabled):hover {
		background: #1b5e20;
		transform: translateY(-1px);
		box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
	}

	.output-container {
		width: 100%;
		max-width: 800px;
		border: 1px solid #e0e0e0;
		border-radius: 6px;
		background: white;
		box-shadow: 0 2px 4px rgba(0, 0, 0, 0.05);
	}

	.output-header {
		display: flex;
		flex-direction: row;
		justify-content: space-between;
		align-items: center;
		gap: 1rem;
		padding: 1rem;
		border-bottom: 1px solid #e0e0e0;
		background: #f1f8e9;
		border-radius: 6px 6px 0 0;
	}

	.output-header h2 {
		margin: 0;
	}

	.copy-button {
		background: #43a047;
		padding: 0 1rem;
		height: 36px;
	}

	.copy-button:hover {
		background: #2e7d32;
	}

	pre {
		background: white;
		padding: 1rem;
		border-radius: 0 0 6px 6px;
		overflow-x: auto;
		margin: 0;
		font-size: 0.85rem;
		line-height: 1.5;
		color: #333;
		white-space: pre-wrap;
		word-break: break-word;
	}

	::-webkit-scrollbar {
		width: 8px;
		height: 8px;
	}

	::-webkit-scrollbar-track {
		background: #f1f8e9;
		border-radius: 4px;
	}

	::-webkit-scrollbar-thumb {
		background: #a5d6a7;
		border-radius: 4px;
	}

	::-webkit-scrollbar-thumb:hover {
		background: #81c784;
	}

	.header-info {
		display: flex;
		flex-direction: column;
		align-items: center;
		gap: 1.25rem;
		margin-bottom: 1rem;
	}

	.author-link {
		color: #2e7d32;
		text-decoration: none;
		font-weight: 500;
	}

	.author-link:hover {
		text-decoration: underline;
	}

	.tools {
		display: flex;
		flex-wrap: wrap;
		justify-content: center;
		gap: 0.5rem;
	}

	.tool-tag {
		display: inline-flex;
		align-items: center;
		padding: 0.5rem 1rem;
		background-color: #f1f8e9;
		color: #2e7d32;
		text-decoration: none;
		border-radius: 9999px;
		font-size: 0.9rem;
		transition: all 0.2s ease;
	}

	.tool-tag:hover {
		background-color: #e8f5e9;
		transform: translateY(-1px);
	}

	@media (max-width: 599px) {
		.input-container {
			width: 100%;
		}

		.main-input-row {
			flex-direction: column;
			width: 100%;
		}

		.input-group {
			flex-direction: column;
			width: 100%;
		}

		.input-wrapper {
			width: 100%;
		}

		.chain-search-wrapper {
			width: 100%;
		}

		.api-key-row {
			flex-direction: column;
			gap: 0.5rem;
		}

		.api-key-row input {
			width: 100%;
		}

		.save-button {
			width: 100%;
		}

		button:not(.copy-button):not(.save-button) {
			width: 100%;
			min-width: unset;
		}

		input,
		.chain-search {
			width: 100%;
			max-width: 100%;
		}

		.header-info {
			margin-bottom: 1rem;
		}
	}

	.info-container {
		padding: 1rem 1.5rem;
		background: #e8f5e9;
		border: 1px solid #a5d6a7;
		border-radius: 6px;
		width: 100%;
		max-width: 800px;
		color: #2e7d32;
		display: flex;
		align-items: center;
		gap: 0.5rem;
	}

	.info-message {
		margin: 0;
		font-size: 0.875rem;
	}

	.contract-info {
		width: 100%;
		max-width: 800px;
		background: #f5f5f5;
		border: 1px solid #e0e0e0;
		border-radius: 8px;
		padding: 1.5rem;
	}

	.contract-info h3 {
		margin: 0 0 1rem 0;
		color: #1b5e20;
		font-size: 1.125rem;
		font-weight: 600;
	}

	.info-grid {
		display: grid;
		grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
		gap: 0.75rem;
	}

	.info-item {
		display: flex;
		flex-direction: column;
		gap: 0.25rem;
	}

	.info-label {
		font-size: 0.75rem;
		color: #666;
		font-weight: 500;
		text-transform: uppercase;
		letter-spacing: 0.5px;
	}

	.info-value {
		font-size: 0.875rem;
		color: #333;
		font-family: 'Fira Mono', monospace;
	}

	.loader-container {
		width: 100%;
		max-width: 800px;
		padding: 1.5rem;
		background: white;
		border: 1px solid #e0e0e0;
		border-radius: 6px;
		box-shadow: 0 2px 4px rgba(0, 0, 0, 0.05);
	}

	.loader {
		display: flex;
		flex-direction: column;
		align-items: center;
		gap: 1rem;
	}

	.loader-text {
		font-size: 1rem;
		color: #2e7d32;
		text-align: center;
	}

	.loader-spinner {
		width: 32px;
		height: 32px;
		border: 3px solid rgba(46, 125, 50, 0.1);
		border-radius: 50%;
		border-top-color: #2e7d32;
		animation: spin 1s ease-in-out infinite;
	}

	@keyframes spin {
		to {
			transform: rotate(360deg);
		}
	}

	.input-with-icon {
		position: relative;
		width: 100%;
	}

	.search-icon {
		position: absolute;
		right: 12px;
		top: 50%;
		transform: translateY(-50%);
		width: 20px;
		height: 20px;
		color: #2e7d32;
		opacity: 0.5;
		pointer-events: none;
	}

	.chain-search {
		padding-right: 40px;
	}
</style>
