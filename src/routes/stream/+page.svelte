<script>
    import { page } from "$app/stores";
    import { onMount } from "svelte";

    let streamLink;

    onMount(() => {
        streamLink = atob($page.url.searchParams.get("stream"));
        const video = document.querySelector("video");
        const source = video.getElementsByTagName("source")[0].src;

        // For more options see: https://github.com/sampotts/plyr/#options
        // captions.update is required for captions to work with hls.js
        const defaultOptions = {};

        if (Hls.isSupported()) {
            // For more Hls.js options, see https://github.com/dailymotion/hls.js
            const hls = new Hls();
            hls.loadSource(source);

            // From the m3u8 playlist, hls parses the manifest and returns
            // all available video qualities. This is important, in this approach,
            // we will have one source on the Plyr player.
            hls.on(Hls.Events.MANIFEST_PARSED, function (event, data) {
                // Transform available levels into an array of integers (height values).
                const availableQualities = hls.levels.map((l) => l.height);

                // Add new qualities to option
                defaultOptions.quality = {
                    default: availableQualities[0],
                    options: availableQualities,
                    // this ensures Plyr to use Hls to update quality level
                    forced: true,
                    onChange: (e) => updateQuality(e),
                };

                defaultOptions.captions = {
                    active: true,
                    language: "auto",
                    update: false,
                };

                // Initialize here
                const player = new Plyr(video, defaultOptions);
            });
            hls.attachMedia(video);
            window.hls = hls;
        } else {
            // default options with no quality update in case Hls is not supported
            const player = new Plyr(video, defaultOptions);
        }

        function updateQuality(newQuality) {
            window.hls.levels.forEach((level, levelIndex) => {
                if (level.height === newQuality) {
                    console.log("Found quality match with " + newQuality);
                    window.hls.currentLevel = levelIndex;
                }
            });
        }
    });
</script>

<svelte:head>
    <title>Stream CCTV!</title>
    <meta name="description" content="Pantau CCTV Seluruh Indonesia!" />
    <link rel="stylesheet" href="https://unpkg.com/plyr@3/dist/plyr.css" />
</svelte:head>
<section class="w-full flex gap-3">
    <div class="w-full md:w-2/3 p-2">
        {#if streamLink}
            <video controls crossorigin playsinline class="w-full">
                <source type="application/x-mpegURL" src={streamLink} />
            </video>
        {:else}
            <p>Loading...</p>
        {/if}
    </div>
    <div class="w-full md:w-1/3 p-2">
        <p>Chat</p>
    </div>
</section>
