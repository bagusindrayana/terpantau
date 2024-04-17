<script>
    import { onMount, onDestroy } from "svelte";
    import { PUBLIC_API_CORS } from "$env/static/public"
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
    function checkSteam(link) {
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

            // For more options see: https://github.com/sampotts/plyr/#options
            // captions.update is required for captions to work with hls.js
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

        // function updateQuality(newQuality) {
        //     window.hls.levels.forEach((level, levelIndex) => {
        //         if (level.height === newQuality) {
        //             console.log("Found quality match with " + newQuality);
        //             window.hls.currentLevel = levelIndex;
        //         }
        //     });
        // }
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
        if(item.header && PUBLIC_API_CORS && !isEmpty(item.header)){
            link = PUBLIC_API_CORS + encodeURIComponent(item.stream) + "?headers=" + encodeURIComponent(JSON.stringify(item.header));
        }
        streamLink = link;
        checkSteam(link);

        setTimeout(() => {
            if (hls) {
                hls.stopLoad();
            }
        }, 5000);
    });

    onDestroy(() => {
        if (player) {
            player.destroy();
        }

        if (hls) {
            hls.stopLoad();
        }
    });
</script>

<div class="w-full lg:w-1/3 sm:w-1/2 p-4">
    <a
        class="flex justify-center relative overflow-hidden"
        href="/stream?stream={btoa(unescape(encodeURIComponent(JSON.stringify(item))))}"
    >
        <!-- <img
            alt="gallery"
            class="absolute inset-0 w-full h-full object-cover object-center"
            src="https://dummyimage.com/600x360"
        /> -->
        <div
            class="absolute flex justify-center inset-0 w-full h-full object-cover object-center bg-gray-300"
        >
            <video id={"video-" + id} crossorigin class=" w-full h-full">
                <source src={streamLink} />
            </video>
        </div>

        <div
            class="px-8 py-10 relative z-10 w-full border-4 border-gray-200 bg-white opacity-0 hover:opacity-100"
        >
            <h2 class="tracking-widest text-sm title-font font-medium mb-1">
                {#if online}
                    <span class="text-green-500">Online</span>
                {:else}
                    <span class="text-red-500">Offline</span>
                {/if}
            </h2>
            <h1 class="title-font text-lg font-medium text-gray-900 mb-3">
                {item.name}
            </h1>
            <p class="leading-relaxed">
                Sumber : {source}
            </p>
        </div>
    </a>
</div>