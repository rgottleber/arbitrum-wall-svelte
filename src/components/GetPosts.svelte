<script>
	export let postContract;
	export let account;
	export let owner;
	$: allPosts = [];
	const cleanPosts = async (posts) => {
		const cleanedPosts = [];
		posts.forEach((post) => {
			if (account !== owner && post.from !== account && post.approved === false) {
				return;
			}
			cleanedPosts.push({
				address: post.from,
				timestamp: new Date(post.timestamp * 1000),
				message: post.message,
				approved: post.approved,
				id: post.id
			});
		});
		return cleanedPosts;
	};

	const getPosts = async () => {
		const posts = await postContract.getAllMessages();
		allPosts = (await cleanPosts(posts)).reverse();
		postContract.on('NewMessage', (from, timestamp, message, approved, id) => {
			allPosts = [
				{
					address: from,
					timestamp: new Date(timestamp * 1000),
					message: message,
					approved: approved,
					id: id
				},
				...allPosts
			];
		});
	};
	const approvePost = async (post) => {
		if (!post.approved) {
			await postContract.approveMessage(post.id);
		}
	};
	getPosts();
</script>

<div class="postitsCollection">
	{#each allPosts as post}
		<div class="postit {post.approved ? '' : 'pending'}" on:click={approvePost(post)}>
			<div class="time">
				{new Date(Date.parse(post.timestamp)).toDateString()}
			</div>
			<div class="message">
				{post.message}
			</div>
			<div class="address">{post.address}</div>
		</div>
	{/each}
</div>

<style>
	.time {
		color: #999;
		position: absolute;
		top: -2em;
	}
	.message {
		position: relative;
		margin-top: -2.5em;
		margin-bottom: 1em;
	}
	.address {
		font-size: 0.7em;
		color: #999;
		position: absolute;
		bottom: 0.75em;
	}
	.postitsCollection {
		display: grid;
		gap: 1em;
		grid-template-columns: repeat(auto-fill, minmax(275px, 1fr));
	}
	.postit {
		color: rgba(0, 0, 0, 0.4);
		line-height: 1;
		text-align: center;
		width: 275px;
		margin: 25px;
		min-height: 250px;
		max-height: 250px;
		padding-top: 35px;
		position: relative;
		border: 1px solid #e8e8e8;
		border-top: 60px solid #fdfd86;
		font-family: 'Reenie Beanie';
		font-size: 22px;
		border-bottom-right-radius: 60px 5px;
		display: grid;
		background: #ffff88; /* Old browsers */
		background: -moz-linear-gradient(
			-45deg,
			#ffff88 81%,
			#ffff88 82%,
			#ffff88 82%,
			#ffffc6 100%
		); /* FF3.6+ */
		background: -webkit-gradient(
			linear,
			left top,
			right bottom,
			color-stop(81%, #ffff88),
			color-stop(82%, #ffff88),
			color-stop(82%, #ffff88),
			color-stop(100%, #ffffc6)
		); /* Chrome,Safari4+ */
		background: -webkit-linear-gradient(
			-45deg,
			#ffff88 81%,
			#ffff88 82%,
			#ffff88 82%,
			#ffffc6 100%
		); /* Chrome10+,Safari5.1+ */
		background: -o-linear-gradient(
			-45deg,
			#ffff88 81%,
			#ffff88 82%,
			#ffff88 82%,
			#ffffc6 100%
		); /* Opera 11.10+ */
		background: -ms-linear-gradient(
			-45deg,
			#ffff88 81%,
			#ffff88 82%,
			#ffff88 82%,
			#ffffc6 100%
		); /* IE10+ */
		background: linear-gradient(
			135deg,
			#ffff88 81%,
			#ffff88 82%,
			#ffff88 82%,
			#ffffc6 100%
		); /* W3C */
	}
	.pending {
		background: #ff8888; /* Old browsers */
		background: -moz-linear-gradient(
			-45deg,
			#ff8888 81%,
			#ff8888 82%,
			#ff8888 82%,
			#ffffc6 100%
		); /* FF3.6+ */
		background: -webkit-gradient(
			linear,
			left top,
			right bottom,
			color-stop(81%, #ff8888),
			color-stop(82%, #ff8888),
			color-stop(82%, #ff8888),
			color-stop(100%, #ff88c6)
		); /* Chrome,Safari4+ */
		background: -webkit-linear-gradient(
			-45deg,
			#ff8888 81%,
			#ff8888 82%,
			#ff8888 82%,
			#ff88c6 100%
		); /* Chrome10+,Safari5.1+ */
		background: -o-linear-gradient(
			-45deg,
			#ff8888 81%,
			#ff8888 82%,
			#ff8888 82%,
			#ff88c6 100%
		); /* Opera 11.10+ */
		background: -ms-linear-gradient(
			-45deg,
			#ff8888 81%,
			#ff8888 82%,
			#ff8888 82%,
			#ff88c6 100%
		); /* IE10+ */
		background: linear-gradient(
			135deg,
			#ff8888 81%,
			#ff8888 82%,
			#ff8888 82%,
			#ff88c6 100%
		); /* W3C */
	}
	.postit:after {
		content: '';
		position: absolute;
		z-index: -1;
		right: -0px;
		bottom: 20px;
		width: 200px;
		height: 25px;
		background: rgba(0, 0, 0, 0.2);
		box-shadow: 2px 15px 5px rgba(0, 0, 0, 0.4);
		-moz-transform: matrix(-1, -0.1, 0, 1, 0, 0);
		-webkit-transform: matrix(-1, -0.1, 0, 1, 0, 0);
		-o-transform: matrix(-1, -0.1, 0, 1, 0, 0);
		-ms-transform: matrix(-1, -0.1, 0, 1, 0, 0);
		transform: matrix(-1, -0.1, 0, 1, 0, 0);
	}
</style>
