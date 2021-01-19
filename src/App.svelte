<script>
	import { onMount } from 'svelte';
	import Loading from './Loading.svelte';
	import About from './About.svelte';
	import Nav from './Nav.svelte';
	const { createFFmpeg, fetchFile } = FFmpeg;
	const ffmpeg = createFFmpeg({log: true});

	let ready = false;
	let success = false
	let error = "";
	let filename = "Drag your video here or click in this area."
	let hover = "";
	let page = "upload"

	onMount(async () => {
		try{
			await ffmpeg.load();
			success = true
		} catch (e) {
			error = e.message;
			success = false;
		}
		ready = true;
	});

	let files = null;
	let gif = null;
	let converting = false;
	let duration = null;

	let width = null;
	let height = null;
	let start = null;
	let end = null;
	let framerate = 30;

	async function convertToGif() {
		gif = null;
		converting = true;
		callback("gif");
		ffmpeg.FS("writeFile", "test.mp4", await fetchFile(files[0]));
		console.log(end-start);
		await ffmpeg.run("-r", `${framerate}`, "-i", "test.mp4", "-t", `${end-start}`, "-ss", `${start}`, "-vf", `scale=${width}:${height}`, "-f", "gif", "out.gif");
		const data = ffmpeg.FS("readFile", "out.gif");
		gif = URL.createObjectURL(new Blob([data.buffer], {type: "image/gif"}));
		converting = false;
		callback("gif");
	}

	function sleep(ms) {
		return new Promise(resolve => setTimeout(resolve, ms));
	}

	async function refresh() {
		width = null;
		height = null;
		hover = "";
		filename = files[0].name;
		callback("video");
	}

	async function drop() {
		filename = files[0].name;
	}

	async function enter() {
		hover = "dropzone";
	}

	async function exit() {
		hover = "";
	}

	async function callback(target) {
		page = target;
		if(target == "video") {
			framerate = 30;
			duration = null;
			while(!duration) {
				let element = document.getElementById("video");
				if(element) {
					duration = Math.round(element.duration * 100) / 100;
				}
				await sleep(50);
			}

			width = null;
			height = null;
			while(!width && !height) {
				let element = document.getElementById("video");
				if(element) {
					width = element.videoWidth;
					height = element.videoHeight;
				}
				await sleep(50);
			}
		}
	}

	$: {
		if(duration) {
			start = 0;
			end = duration;
		}
	}
</script>

<main>
	<Nav page={page} video={files} gif={gif} callback={callback}/>
	<div id="content">
		{#if ready}
			{#if success}
				{#if page == "upload"}
					<section>
						<input type="file" bind:files on:change={refresh} on:drop={drop} on:dragover={enter} on:dragleave={exit}>
						<h2 class={hover}>{filename}</h2>
					</section>
				{:else if page == "video"}
					<form on:submit|preventDefault={convertToGif}>
						<video id="video" controls=true src={URL.createObjectURL(files[0])}></video>
						<div id="buttons">
							<div class="button">
								<label for="start">Start</label>
								<input min="0" max={end} step="0.01" name="start" type="number" bind:value={start}>
							</div>
							<div class="button">
								<label for="end">End</label>
								<input step="0.01" max={duration} name="end" type="number" bind:value={end}>
							</div>
							<div class="button">
								<label for="width">Width</label>
								<input min="0" step="0.01" name="width" type="number" bind:value={width}>
							</div>
							<div class="button">
								<label for="height">Height</label>
								<input min="0" step="0.01"  name="height" type="number" bind:value={height}>
							</div>
							<div class="button">
								<label for="framerate">Framerate</label>
								<input min="0" step="0.01" name="framerate" type="number" bind:value={framerate}>
							</div>
							<button disabled={converting}>Convert!</button>
						</div>
					</form>
				{:else if page == "gif"}
					{#if !converting}
						<img id="gif" src={gif}>
					{:else}
						<Loading/>
					{/if}
				{:else if page == "about"}
					<About/>
				{/if}
			{:else}
				<h1>Error: {error}</h1>
				<p>If you get an error it means this web page uses features your browser does not support.</p>
			{/if}
		{:else}
			<Loading/>
		{/if}
	</div>
</main>

<style>
	img {
		width: 700px;
	}

	video {
		width: 700px;
		margin-bottom: 10px;
	}

	#content {
		display: flex;
		justify-content: center;
		align-items: center;
		height: calc(100vh - 63px);
		padding: 0px;
		margin: 0px;
	}

	#buttons {
		display: flex;
	}

	section {
		height: 70vh;
		width: 60vw;
		border: 6px dashed white;
		display: flex;
		justify-content: center;
		align-items: center;
	}

	input[type="number"] {
		width: 80px;
		border-radius: 7px;
		border: solid 1px #30363D;
		background-color: #090D13;
		color: white;
		font-size: 14px;
        padding: 5px 12px;
        line-height: 20px;
	}

	input:focus{
        outline: none;
    }

    input:invalid {
        box-shadow: none;
	}
	
	label {
		font-weight: 800px;
		font-size: 14px;
		margin-bottom: 7px;
    }

	input[type="file"] {
		position: absolute;
		height: 70vh;
		width: 60vw;
		margin: 0;
		padding: 0;
		outline: none;
		opacity: 0;
		-moz-appearance: textfield;
	}

	input::-webkit-inner-spin-button {
		-webkit-appearance: none;
		margin: 0;
	}

	h2 {
		text-align: center;
		color: white;
		font-weight: bold;
		font-size: 30px;
		margin: 20px;
	}

	.dropzone {
		color: var(--main-color);
	}

	button {
		display: block;
	}

	p {
		margin-right: 20px;
	}

	.button {
		margin-right: 30px;
	}
</style>