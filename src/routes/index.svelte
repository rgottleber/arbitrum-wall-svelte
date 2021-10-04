<script>
	import { onMount } from 'svelte';
	import { ethers } from 'ethers';
	import GetPosts from '../components/GetPosts.svelte';
	import MakePost from '../components/MakePost.svelte';
	$: account = null;
	let web3Props;

	onMount(async () => {
		try {
			const { ethereum } = window;
			const provider = new ethers.providers.Web3Provider(ethereum);
			const signer = provider.getSigner();
			const network = await provider.getNetwork();
			const networkName = network.name;
			const contractAddress = '0x79d4A810f41F09e8aa3e1daa560c648E6eD43833';
			web3Props = {
				provider,
				signer,
				network,
				networkName,
				contractAddress
			};
			if (!ethereum) {
				alert('Get MetaMask!');
				return;
			}
			const accounts = await ethereum.request({ method: 'eth_accounts' });
			if (accounts.length !== 0) {
				account = accounts[0];
			}
		} catch (error) {
			console.log(error);
		}
	});
	async function attachWallet() {
		const provider = new ethers.providers.Web3Provider(window.ethereum, 'any');
		// Prompt user for account connections
		await provider.send('eth_requestAccounts', []);
		const signer = provider.getSigner();
		account = await signer.getAddress();
	}
</script>

<h1>✍️ Post something on my 📓 wall</h1>

{#if !account}
	<p>This application is build on the Arbitrum Test Network.</p>
	<p>
		<a href="https://arbitrum.io/">
			<img
				src="https://arbitrum.io/wp-content/uploads/2021/01/cropped-Arbitrum_Horizontal-Logo-Full-color-White-background-scaled-1.jpg"
				alt="Arbitrum"
				width="100"
			/>
		</a>
	</p>
	<p>
		<button on:click={attachWallet}>Attach Wallet</button>
	</p>
{:else}
	<MakePost {...web3Props} />
	<GetPosts {...web3Props} />
{/if}

<style>
	h1 {
		text-align: center;
	}
	p {
		text-align: center;
	}
</style>
