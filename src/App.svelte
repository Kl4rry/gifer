<script>
	import { onMount } from 'svelte';
	import Loading from './Loading.svelte';
	const { createFFmpeg, fetchFile } = FFmpeg;
	const ffmpeg = createFFmpeg({log: true});

	let ready = false;
	let success = false

	onMount(async () => {
		try{
			await ffmpeg.load();
			success = true
		} catch (e) {
			if(e instanceof ReferenceError) {
				success = false;
			}
		}
		ready = true;
	});

	let files = null;
	let gif = null;
	let converting = false;
	let duration = null;

	async function convertToGif(){
		console.log(files[0]);
		converting = true;
		ffmpeg.FS("writeFile", "test.mp4", await fetchFile(files[0]));
		await ffmpeg.run("-i", "test.mp4","-f", "gif", "out.gif");
		const data = ffmpeg.FS("readFile", "out.gif");
		gif = URL.createObjectURL(new Blob([data.buffer], {type: "image/gif"}))
		converting = false;
	}

	function sleep(ms) {
		return new Promise(resolve => setTimeout(resolve, ms));
	}

	async function refresh(){
		duration = document.getElementById("video").duration; 
		while(!duration){
			await sleep(50);
			duration = document.getElementById("video").duration; 
		}
	}
</script>

<main>
	{#if ready}
		{#if success}
			<h1>Ready</h1>
			<input type="file" bind:files on:change={refresh}>
			{#if files}
				<video id="video" controls=true src={URL.createObjectURL(files[0])}></video>
				<button disabled={converting} on:click={convertToGif}>Convert!</button>
				<p>{duration}</p>
			{/if}
			{#if files != null}
				<img src={gif}>
			{/if}
		{:else}
			<h1>This browser does not support SharedArrayBuffer</h1>
		{/if}
	{:else}
		<Loading/>
	{/if}
</main>

<style>
	#img{
		width: 500px;
	}
	#video{
		width: 500px;
	}
</style>