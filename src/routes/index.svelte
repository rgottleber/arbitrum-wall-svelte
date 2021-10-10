<script>
	import { onMount } from 'svelte';
	import { ethers } from 'ethers';
	import Wall from '../contracts/Wall.json';
	import GetPosts from '../components/GetPosts.svelte';
	import MakePost from '../components/MakePost.svelte';
	$: account = null;
	$: chainID = null;
	let web3Props;
	onMount(async () => {
		try {
			const { ethereum } = window;
			const provider = new ethers.providers.Web3Provider(ethereum);
			const signer = provider.getSigner();
			account = await signer.getAddress();
			const network = await provider.getNetwork();
			const networkName = network.name;
			// const contractAddress = '0x79d4A810f41F09e8aa3e1daa560c648E6eD43833';
			const contractAddress = '0x1425B5c98E9690472643448c071137bf131C0732';
			const postContract = new ethers.Contract(contractAddress, Wall.abi, signer);
			const owner = await postContract.owner();

			web3Props = {
				postContract,
				account,
				owner
			};
			if (!ethereum) {
				alert('Get MetaMask!');
				return;
			}
			const accounts = await ethereum.request({ method: 'eth_accounts' });
			if (accounts.length !== 0) {
				account = accounts[0];
				chainID = await signer.getChainId();
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
		chainID = await signer.getChainId();
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
{:else if chainID != 421611}
	<h3>Please use the Arbitrum TestNet</h3>
	<p>Network Name: Arbitrum Testnet</p>
	<p>New RPC URL: https://rinkeby.arbitrum.io/rpc</p>
	<p>Chain ID: 421611</p>
	<p>Currency Symbol: ETH</p>
	<p>Block Explorer URL: https://rinkeby.arbitrum.io/#/</p>
{:else}
	<MakePost {...web3Props} />
	<GetPosts {...web3Props} />
{/if}

<style>
	h1,
	h3,
	p {
		text-align: center;
	}
</style>
