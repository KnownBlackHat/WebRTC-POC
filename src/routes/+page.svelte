<script lang="ts">
	import { onMount } from 'svelte';

	let localStream: MediaStream;
	let remoteStream: MediaStream;

	let localVid: HTMLVideoElement;
	let remoteVid: HTMLVideoElement;

	let offer: string;
	let answer: string;
	let pc: RTCPeerConnection;

	let error: string;

	const servers = {
		iceServers: [
			{
				urls: ['stun:stun1.l.google.com:19302', 'stun:stun2.l.google.com:19302']
			}
		]
	};

	onMount(async () => {
		localStream = new MediaStream();
		localVid.srcObject = localStream;
	});

	const createPeerConnection = async () => {
		remoteStream = new MediaStream();
		remoteVid.srcObject = remoteStream;

		pc = new RTCPeerConnection(servers);

		localStream.getTracks().forEach((track) => {
			pc.addTrack(track, localStream);
		});

		pc.ontrack = (event) => {
			event.streams[0].getTracks().forEach((track) => {
				remoteStream.addTrack(track);
			});
		};
	};

	const createOffer = async () => {
		await createPeerConnection();
		let offer_desc = await pc.createOffer();
		pc.setLocalDescription(offer_desc);
		offer = JSON.stringify(offer_desc);
		pc.onicecandidate = async () => {
			offer = JSON.stringify(pc.localDescription);
		};
	};

	const createAnswer = async () => {
		await createPeerConnection();
		await pc.setRemoteDescription(JSON.parse(offer));
		let answer_desc = await pc.createAnswer();
		await pc.setLocalDescription(answer_desc);
		answer = JSON.stringify(answer_desc);
		pc.onicecandidate = async () => {
			offer = JSON.stringify(pc.localDescription);
		};
	};

	const addAnswer = async () => {
		await pc.setRemoteDescription(JSON.parse(answer));
	};
</script>

<svelte:head>
	<title>WebRTC Concept Model</title>
</svelte:head>

<div class="text-2xl text-center">WebRTC Concept Model</div>
<div class="flex justify-center">
	<video
		playsinline
		autoplay
		bind:this={localVid}
		class="border-2 border-red-50
    bg-black m-4 h-[30rem] w-[40rem]"
	>
		<track kind="captions" />
	</video>
	<video
		playsinline
		autoplay
		bind:this={remoteVid}
		class="border-2 border-red-50
    bg-black m-4 h-[30rem] w-[40rem]"
	>
		<track kind="captions" />
	</video>
</div>

{#if error}
	<div class="text-red-500 text-center">
		{error}
	</div>
{:else}
	<div class="flex justify-center items-center">
		<button
			class="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded"
			on:click={createOffer}
		>
			Create Offer
		</button>
		<textarea
			bind:value={offer}
			class="border-2 border-red-500 m-4 h-12 w-full bg-slate-700 rounded-lg"
		/>

		<button
			class="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded"
			on:click={createAnswer}
		>
			Create Answer
		</button>
		<textarea
			bind:value={answer}
			class="border-2 border-red-500 m-4 h-12 w-full bg-slate-700 rounded-lg"
		/>

		<button
			class="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded"
			on:click={addAnswer}
		>
			Add Answer
		</button>
	</div>
{/if}
<div class="text-center space-x-2">
	Media:
	{#each ['screen', 'camera'] as media}
		<input
			type="radio"
			name="media"
			class="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded"
			on:click={async () => {
				try {
					if (media === 'screen') {
						localStream = await navigator.mediaDevices.getDisplayMedia();
					} else {
						localStream = await navigator.mediaDevices.getUserMedia({ video: true, audio: true });
					}
				} catch (err) {
					error = err;
				}
				localVid.srcObject = localStream;
			}}
		/>
		<label for={media}> {media.toUpperCase()} </label>
	{/each}
</div>
