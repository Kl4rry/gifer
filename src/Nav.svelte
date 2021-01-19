<script>
    export let page;
    export let video;
    export let gif;
    export let callback;

    async function onUpload() {
        callback("upload");
    }

    async function onVideo() {
        callback("video");
    }

    async function onGif() {
        callback("gif");
    }

    async function onAbout() {
        callback("about");
    }
</script>

<nav>
	<ul>
        <li class="left"><a aria-current="{page === "upload" ? "page" : undefined}" on:click={onUpload}>Upload</a></li>
        {#if video}
            <li class="left"><a aria-current="{page === "video" ? "page" : undefined}" on:click={onVideo}>Video</a></li>
        {/if}
        {#if gif}
            <li class="left"><a aria-current="{page === "gif" ? "page" : undefined}" on:click={onGif}>Gif</a></li>
        {/if}
        <li class="right"><a aria-current="{page === "about" ? "page" : undefined}" on:click={onAbout}>About</a></li>
	</ul>
</nav>

<style>
	nav {
		background-color: #161B22;
		color: white;
		padding-left: 10px;
		padding-right: 20px;
		height: 63px;
	}

	ul {
		margin: 0;
		padding: 0;
	}

	ul::after {
		content: "";
		display: block;
		clear: both;
	}

	li {
		display: block;
		font-size: 20px;
		font-weight: 600;
	}

	.left {
		float: left;
    }
    
    .right {
		float: right;
	}

	[aria-current] {
		position: relative;
		display: inline-block;
	}

	[aria-current]::after {
		position: absolute;
		content: "";
		width: calc(100% - 1em);
		height: 2px;
		background-color: rgb(255,62,0);
		display: block;
		bottom: -1px;
	}

	a {
		text-decoration: none;
		padding: 1em 0.5em;
        display: block;
        color: white;
    }
    a:hover {
        cursor: pointer;
    }
</style>
