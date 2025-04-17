<script lang="ts">
	import { createPublicClient, http, isAddress } from 'viem';
	import * as chains from 'viem/chains';
	import { whatsabi } from '@shazow/whatsabi';

	// Click outside action
	function clickOutside(node: HTMLElement, handler: () => void) {
		const handleClick = (event: MouseEvent) => {
			if (!node.contains(event.target as Node)) {
				handler();
			}
		};

		document.addEventListener('click', handleClick, true);

		return {
			destroy() {
				document.removeEventListener('click', handleClick, true);
			}
		};
	}

	// Convert chains object to array and filter out non-chain exports
	const allChains = Object.entries(chains)
		.filter(([_, value]) => value && typeof value === 'object' && 'id' in value)
		.map(([key, chain]) => ({
			name: key
				.replace(/([A-Z])/g, ' $1') // Add spaces before capital letters
				.trim()
				.split(' ')
				.map((word) => word.charAt(0).toUpperCase() + word.slice(1)) // Capitalize each word
				.join(' '),
			chain
		}))
		.sort((a, b) => a.name.localeCompare(b.name));

	const mainnetIndex = allChains.findIndex((chain) => {
		if (chain.chain.id === 1) {
			return chain;
		}
	});

	allChains[mainnetIndex].name = 'Ethereum Mainnet';

	let abi: any[] = [];
	let address = '';
	let chainSearch = '';
	let copySuccess = false;
	let addressError = '';
	let fetchError = '';
	let selectedChain = allChains[0];
	let showChainDropdown = false;
	let isLoading = false;
	let proxyNotice = '';

	$: filteredChains = chainSearch
		? allChains.filter((chain) => chain.name.toLowerCase().includes(chainSearch.toLowerCase()))
		: allChains;

	function selectChain(chain: (typeof allChains)[0]) {
		selectedChain = chain;
		chainSearch = chain.name;
		showChainDropdown = false;
	}

	function closeDropdown() {
		showChainDropdown = false;
	}

	function validateAddress() {
		if (!address) {
			addressError = 'Address is required';
			return false;
		}
		if (!isAddress(address)) {
			addressError = 'Invalid Ethereum address';
			return false;
		}
		addressError = '';
		return true;
	}

	async function getABI() {
		if (!validateAddress()) return;

		isLoading = true;
		fetchError = '';
		abi = [];

		try {
			const client = createPublicClient({
				chain: selectedChain.chain,
				transport: http()
			});

			// Check if address is a contract
			const bytecode = await client.getCode({ address: address as `0x${string}` });
			if (!bytecode) {
				fetchError = 'This address is not a contract';
				isLoading = false;
				return;
			}

			const { abi: resolvedABI, address: resolvedAddress } = await whatsabi.autoload(address, {
				provider: client,
				followProxies: true
			});
			abi = resolvedABI;

			if (resolvedAddress.toLowerCase() !== address.toLowerCase()) {
				proxyNotice = `Note: This is a proxy contract. The actual implementation address is ${resolvedAddress}`;
			} else {
				proxyNotice = '';
			}
			console.log('Implementation Address', resolvedAddress);
		} catch (error) {
			console.error('Error fetching ABI:', error);
			fetchError = error instanceof Error ? error.message : 'Failed to fetch ABI';
		} finally {
			isLoading = false;
		}
	}

	async function copyABI() {
		try {
			await navigator.clipboard.writeText(JSON.stringify(abi, null, 2));
			copySuccess = true;
			setTimeout(() => (copySuccess = false), 2000);
		} catch (error) {
			console.error('Error copying to clipboard:', error);
		}
	}

	function handleInput() {
		if (address) validateAddress();
		else addressError = '';
	}
</script>

<svelte:head>
	<title>Immortal WhatsABI</title>
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
				Viem v2.27.2
			</a>
			<a
				href="https://github.com/shazow/whatsabi"
				target="_blank"
				rel="noopener noreferrer"
				class="tool-tag"
			>
				WhatsABI v0.21.0
			</a>
			<a href="https://fleek.xyz" target="_blank" rel="noopener noreferrer" class="tool-tag">
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
				Svelte v4.2.12
			</a>
		</div>
	</div>
	<div class="input-container">
		<div class="input-group">
			<div class="input-wrapper">
				<input
					type="text"
					bind:value={address}
					on:input={handleInput}
					placeholder="Enter contract address"
					class:error={addressError}
				/>
				{#if addressError}
					<span class="error-message">{addressError}</span>
				{/if}
			</div>
			<div class="chain-search-wrapper" use:clickOutside={closeDropdown}>
				<input
					type="text"
					bind:value={chainSearch}
					on:focus={() => (showChainDropdown = true)}
					placeholder="Search chain..."
					class="chain-search"
				/>
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
		<button on:click={getABI} disabled={isLoading}>
			{#if isLoading}
				Loading...
			{:else}
				Get ABI
			{/if}
		</button>
	</div>

	{#if fetchError}
		<div class="error-container">
			<p class="error-message">{fetchError}</p>
		</div>
	{/if}

	{#if proxyNotice}
		<div class="info-container">
			<p class="info-message">{proxyNotice}</p>
		</div>
	{/if}

	{#if abi.length > 0}
		<div class="output-container">
			<div class="output-header">
				<h2>Contract ABI</h2>
				<button class="copy-button" on:click={copyABI}>
					{copySuccess ? 'Copied!' : 'Copy'}
				</button>
			</div>
			<pre>{JSON.stringify(abi, null, 2)}</pre>
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
			flex-direction: row;
			align-items: flex-start;
			max-width: 800px;
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
		flex-direction: row;
		gap: 1rem;
		align-items: flex-start;
		width: 100%;
		max-width: 800px;
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

		button:not(.copy-button) {
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
</style>
