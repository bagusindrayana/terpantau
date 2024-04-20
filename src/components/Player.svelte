<script>
    import { onMount,onDestroy } from "svelte";
    import Plyr from "plyr";
    import Hls from "hls.js";
    import { PUBLIC_API_CORS } from "$env/static/public";

    /**
     * @type {boolean}
     */
    export let preview = false;

    /**
     * @type {Object}
     */
    export let streamInfo;

    /**
     * @type {string}
     */
    let streamLink;

    /**
     * @type {Plyr}
     */
    let plyr;

    /**
     * @type {Hls}
     */
    let hls;

    /**
     * @type {Object}
     */
    let flvPlayer;


    export let id;

    const initVideo = () => {
        console.log(id);
        const video = document.getElementById("video-"+id);
        console.log(video);
        if (video) {
            const source = video.getElementsByTagName("source")[0].src;
            console.log("source", source);
            if (streamInfo.type == "stream") {
                // For more options see: https://github.com/sampotts/plyr/#options
                // captions.update is required for captions to work with hls.js
                const defaultOptions = {};

                if (Hls.isSupported()) {
                    // For more Hls.js options, see https://github.com/dailymotion/hls.js
                    hls = new Hls();
                    try {
                        hls.loadSource(source);
                        // From the m3u8 playlist, hls parses the manifest and returns
                        // all available video qualities. This is important, in this approach,
                        // we will have one source on the Plyr player.
                        hls.on(
                            Hls.Events.MANIFEST_PARSED,
                            function (event, data) {
                                // Transform available levels into an array of integers (height values).
                                const availableQualities = hls.levels.map(
                                    (l) => l.height,
                                );

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
                                plyr = new Plyr(video, defaultOptions);
                            },
                        );

                        hls.on(Hls.Events.ERROR, function () {
                            console.log("video play error");
                            // stopAll();
                        });
                        hls.attachMedia(video);
                        window.hls = hls;
                    } catch (error) {
                        console.log("video load error");
                    }
                } else {
                    // default options with no quality update in case Hls is not supported
                    plyr = new Plyr(video, defaultOptions);
                }
            } else if (streamInfo.type == "video") {
                //extension check
                const ext = source.split(".").pop();
                if (ext == "flv") {
                    if (flvjs.isSupported()) {
                        video.getElementsByTagName("source")[0].remove();
                        flvPlayer = flvjs.createPlayer({
                            type: "flv",
                            isLive: true,
                            hasAudio: false,
                            url: source,
                        });
                        flvPlayer.attachMediaElement(video);
                        flvPlayer.load();
                        // flvPlayer.play();
                    }
                }
            }
        }
    };

    const stopAll = () => {
        if (hls) {
            hls.stopLoad();
            hls.destroy();
        }
        if (plyr) {
            plyr.stop();
        }
        if (flvPlayer) {
            flvPlayer.unload();
            flvPlayer.detachMediaElement();
            flvPlayer.destroy();
        }
    };

    /**
     *
     * @param newQuality {number}
     */
    function updateQuality(newQuality) {
        window.hls.levels.forEach((level, levelIndex) => {
            if (level.height === newQuality) {
                console.log("Found quality match with " + newQuality);
                window.hls.currentLevel = levelIndex;
            }
        });
    }

    /**
     *
     * @param obj {Object}
     * @returns {boolean}
     */
    function isEmpty(obj) {
        for (var prop in obj) {
            if (Object.prototype.hasOwnProperty.call(obj, prop)) {
                return false;
            }
        }

        return true;
    }
    // $: streamLink, initVideo();
    onMount(() => {
        console.log(document.getElementById("video-"+id));
        let link = streamInfo.stream;
        if (
            streamInfo.header &&
            PUBLIC_API_CORS &&
            !isEmpty(streamInfo.header)
        ) {
            link =
                PUBLIC_API_CORS +
                encodeURIComponent(streamInfo.stream) +
                "&headers=" +
                encodeURIComponent(JSON.stringify(streamInfo.header));
                
        }
        streamLink = link;
        
        setTimeout(() => {
            initVideo();
        }, 1000);

        // if (preview) {
        //     setTimeout(() => {
        //         stopAll();
        //     }, 10000);
        // }
    });

    onDestroy(() => {
        stopAll();
    });
</script>

<div class="w-full h-full relative flex justify-center items-center">
    {#if streamInfo != undefined && streamInfo != null && streamLink != undefined && streamLink != null}
        <!-- {#if streamInfo.type == "image"}
            <img src={streamLink} alt="stream image" class="w-full h-full" />
        {:else}
            <video id="video-{id}"
                controls
                crossorigin="true"
                playsinline
                class="w-full h-full"
            >
                <source src={streamLink} />
            </video>
        {/if} -->
        <img src="http://localhost:8111/get?video_url={encodeURIComponent(streamLink)}" class="w-full h-full"/>
    {:else}
        <p>Loading...</p>
    {/if}
</div>
