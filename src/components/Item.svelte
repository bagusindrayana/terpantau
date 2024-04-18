<script>
    import { onMount, onDestroy } from "svelte";
    import { PUBLIC_API_CORS } from "$env/static/public";
    import Plyr from "plyr";
    import Hls from "hls.js";
    /**
     * Represents a item with name and stream attribute.
     * @type {Object}
     */
    export let item;

    /**
     * Represents a item with name and stream attribute.
     * @type {string|null}
     */
    let streamLink = item.stream;

    /**
     * Represents a item with name and stream attribute.
     * @type {string|null}
     */
    export let source;

    /**
     * @type {Plyr}
     */
    let player;

    let flvPlayer;

    /**
     * @type {Hls}
     */
    let hls;

    /**
     * @type {boolean}
     */
    let online = false;

    const id = generateUUID();

    function generateUUID() {
        // Public Domain/MIT
        var d = new Date().getTime(); //Timestamp
        var d2 =
            (typeof performance !== "undefined" &&
                performance.now &&
                performance.now() * 1000) ||
            0; //Time in microseconds since page-load or 0 if unsupported
        return "xxxxxxxx-xxxx-4xxx-yxxx-xxxxxxxxxxxx".replace(
            /[xy]/g,
            function (c) {
                var r = Math.random() * 16; //random number between 0 and 16
                if (d > 0) {
                    //Use timestamp until depleted
                    r = (d + r) % 16 | 0;
                    d = Math.floor(d / 16);
                } else {
                    //Use microseconds since page-load if supported
                    r = (d2 + r) % 16 | 0;
                    d2 = Math.floor(d2 / 16);
                }
                return (c === "x" ? r : (r & 0x3) | 0x8).toString(16);
            },
        );
    }

    var isStream = false;

    /**
     *
     * @param link {string}
     */
    function checkStream(link) {
        //fetch anc cehcn if content type is application/octet-stream or not
        fetch(link, {
            method: "GET",
        }).then((response) => {
            if (response.status == 200) {
                online = true;
                if (response.headers.get("content-type")?.includes("video/")) {
                    isStream = false;
                } else {
                    isStream = true;
                    initVideo();
                }
            } else {
                online = false;
                isStream = false;
            }
        });
    }

    const initVideo = () => {
        /**
         * @type {HTMLVideoElement|null}
         */
        const video = document.querySelector("#video-" + id);

        if (video) {
            const src = video.getElementsByTagName("source")[0].src;
            //extension check
            const ext = src.split(".").pop();
            if (ext == "flv") {
                if (flvjs.isSupported()) {
                    video.getElementsByTagName("source")[0].remove();
                    flvPlayer = flvjs.createPlayer({
                        type: "flv",
                        isLive: true,
                        hasAudio: false,
                        url: src,
                    });
                    flvPlayer.attachMediaElement(video);
                    flvPlayer.load();
                    // flvPlayer.play();
                }
            } else {
                const defaultOptions = {};

                if (Hls.isSupported() && isStream) {
                    // For more Hls.js options, see https://github.com/dailymotion/hls.js
                    hls = new Hls();
                    hls.loadSource(src);

                    // From the m3u8 playlist, hls parses the manifest and returns
                    // all available video qualities. This is important, in this approach,
                    // we will have one source on the Plyr player.
                    hls.on(Hls.Events.MANIFEST_PARSED, function (event, data) {
                        player = new Plyr(video, defaultOptions);
                    });
                    hls.attachMedia(video);

                    // window.hls = hls;
                } else {
                    // default options with no quality update in case Hls is not supported
                    player = new Plyr(video, defaultOptions);
                }
            }
        }
    };

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

    onMount(() => {
        let link = item.stream;
        if (item.header && PUBLIC_API_CORS && !isEmpty(item.header)) {
            link =
                PUBLIC_API_CORS +
                encodeURIComponent(item.stream) +
                "?headers=" +
                encodeURIComponent(JSON.stringify(item.header));
        }
        streamLink = link;
        checkStream(link);

        setTimeout(() => {
            // if (hls) {
            //     hls.stopLoad();
            // }
            checkStream(link);
        }, 500);
    });

    onDestroy(() => {
        if (player) {
            player.destroy();
        }

        if (hls) {
            hls.stopLoad();
        }

        if (flvPlayer) {
            flvPlayer.unload();
            flvPlayer.detachMediaElement();
            flvPlayer.destroy();
        }
    });
</script>

<a
    href="/stream?stream={btoa(
        unescape(encodeURIComponent(JSON.stringify(item))),
    )}"
    class="w-full rounded overflow-hidden shadow-lg"
>
    <div
        class=" w-full h-60 relative flex justify-center items-center bg-gray-500"
    >
        <video id={"video-" + id} crossorigin class=" w-full h-full">
            <source src={streamLink} />
        </video>
    </div>
    <div class="w-full p-4">
        <div class="px-6 py-4">
            <div class="font-bold text-xl mb-2">{item.name}</div>
            <p class="leading-relaxed">
                Sumber : {source}
            </p>
        </div>
        <div class="px-6 pt-4 pb-2">
            {#if online}
                <span class="text-green-500">Online</span>
            {:else}
                <span class="text-red-500">Offline</span>
            {/if}
        </div>
    </div>
</a>
