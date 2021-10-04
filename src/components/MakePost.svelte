<script>
	export let provider;
	export let signer;
	export let network;
	export let networkName;
	export let contractAddress;
	import Wall from '../contracts/Wall.json';
	import { ethers } from 'ethers';
	$: message = 'Write your message here 📝';
	$: mining = false;
	const post = async () => {
		try {
			const { ethereum } = window;
			if (ethereum) {
				const wallContract = new ethers.Contract(contractAddress, Wall.abi, signer);
				const owner = await wallContract.owner();
				const signerAddress = await signer.getAddress();
				let value;
				if (owner === signerAddress) {
					value = 0;
				} else {
					value = ethers.utils.parseEther('0.01');
				}
				const postTxn = await wallContract.post(message, { value: value });

				mining = true;
				await postTxn.wait();
				mining = false;
			} else {
			}
		} catch (error) {
			console.log(error);
		}
	};
</script>

{#if !mining}
	<p>
		<textarea bind:value={message} maxlength="240" />
	</p>
	<p>
		<button on:click={post}>Click to leave a 🗒️ </button>
	</p>
{:else}
	<p>🤖 is currently mining...</p>
	<h3>
		<span class="mining">⛏️</span>
	</h3>
{/if}

<style>
	.mining {
		animation: spin 1s linear infinite;
	}
	@keyframes spin {
		from {
			transform: rotate(0deg);
		}
		to {
			transform: rotate(360deg);
		}
	}
	textarea {
		width: 40%;
		height: 100px;
		border: 1px solid #ccc;
		padding: 10px;
		font-size: 16px;
	}
	p {
		text-align: center;
	}
	button {
		background: var(--primary);
		color: white;
		padding: 10px;
		border: none;
		border-radius: 5px;
	}
</style>
