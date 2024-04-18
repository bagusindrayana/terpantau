<script>
    import { page } from "$app/stores";
    import { onMount } from "svelte";
    import Plyr from "plyr";
    import Hls from "hls.js";
    import { PUBLIC_API_CORS,PUBLIC_API_LIVECHAT } from "$env/static/public";
    import Pusher from "pusher-js";

    var streamInfo = JSON.parse(atob($page.url.searchParams.get("stream")));
    var isStream = false;
    /**
     * Represents a item with name and stream attribute.
     * @type {string|null}
     */
    let streamLink = streamInfo.stream;

    /**
     * Represents a chat message.
     * @type {Array<Object>}
     */
    let chatMessages = [];

    /**
     *
     * @param link {string}
     */
    function checkSteam(link) {
        //fetch anc cehcn if content type is application/octet-stream or not
        fetch(link, {
            method: "GET",
        }).then((response) => {
            if (response.headers.get("content-type")?.includes("video/")) {
                isStream = false;
            } else {
                isStream = true;
                initVideo();
            }
        });
    }

    const initVideo = () => {
        const video = document.querySelector("video");
        if (video) {
            const source = video.getElementsByTagName("source")[0].src;

            // For more options see: https://github.com/sampotts/plyr/#options
            // captions.update is required for captions to work with hls.js
            const defaultOptions = {};

            if (Hls.isSupported() && isStream) {
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
        }

        function updateQuality(newQuality) {
            window.hls.levels.forEach((level, levelIndex) => {
                if (level.height === newQuality) {
                    console.log("Found quality match with " + newQuality);
                    window.hls.currentLevel = levelIndex;
                }
            });
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

    //generate channel name based streamInfo.stream, remove http and symbol

    /**
     *
     * @param e {Event}
     */
    function submitChat() {
        const cm = document.getElementById("chat-message");
        if (cm) {
            const url = PUBLIC_API_LIVECHAT+"/send-message";
            fetch(url, {
                method: "POST",
                headers: {
                    "Content-Type": "application/json",
                    // "Origin": "http://localhost:5197",
                    // "Referer": "http://localhost:5197",
                },
                body: JSON.stringify({
                    message: cm.value,
                    channel:
                        "stream-" +
                        streamInfo.stream.replace(/[^a-zA-Z0-9]/g, ""),
                }),
            }).then((response) => {
                console.log(response);
            });
            cm.value = "";
        }
    }

    onMount(() => {
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
        checkSteam(streamLink);

        var pusher = new Pusher("3d6be57188b3295f4ddd", {
            cluster: "ap1",
        });

        var channel = pusher.subscribe(
            "stream-" + streamInfo.stream.replace(/[^a-zA-Z0-9]/g, ""),
        );
        /**
         * @param data {Object}
         */
        channel.bind("message", function (data) {
            chatMessages.push(data.message);
            chatMessages = chatMessages;
            //scroll to bottom list-chats
            var element = document.getElementById("list-chats");
            setTimeout(() => {
                element.scrollTop = element.scrollHeight;
            }, 200);
        });
    });
</script>

<svelte:head>
    <title>Pantau CCTV - {streamInfo?.name}</title>
    <meta name="description" content={streamInfo?.name} />
    <link rel="stylesheet" href="https://unpkg.com/plyr@3/dist/plyr.css" />
</svelte:head>
<section class="w-full p-2">
    <p>{streamInfo?.name}</p>
</section>
<section class="w-full flex flex-col md:flex-row gap-3" style="max-height: 70vh;">
    <div class="w-full h-full relative md:w-2/3 p-2">
        {#if streamInfo != undefined && streamInfo != null}
            <video controls crossorigin playsinline class="w-full h-1/3 md:h-full video-stream-detail">
                <source src={streamLink} />
            </video>
        {:else}
            <p>Loading...</p>
        {/if}
        <!-- <video name="media" class="stream__frame" id="video_stream" autoplay muted playsinline poster="https://vmssamarinda.iconpln.co.id:8443/mt/api/rest/v1/media?session=AYMAfP8KEOFwg8Ds1UExjWk1VvcCCekSFgoU5v9_dWIirrX3qCM7NVtkQOoF1zQaEAK1f5vl_EV7ssdxdYw0vdMiEG4SCpxUgk9ivN4Jozv2qgkqEFBlbWtvdCBTYW1hcmluZGEyIRoQTZ6shHLARv-_iI3yRVfLVyoNYWRtaW5pc3RyYXRvcg&amp;cameraId=4xIx1DOxSE00M040NE9LMddLTsw1MBASEE7vKs8odvBuM3dQEV72LA4A&amp;format=jpeg&amp;frames=all&amp;media=video&amp;quality=high&amp;t=live" src="https://vmssamarinda.iconpln.co.id:8443/mt/api/rest/v1/media?session=AYMAfP8KEOFwg8Ds1UExjWk1VvcCCekSFgoU5v9_dWIirrX3qCM7NVtkQOoF1zQaEAK1f5vl_EV7ssdxdYw0vdMiEG4SCpxUgk9ivN4Jozv2qgkqEFBlbWtvdCBTYW1hcmluZGEyIRoQTZ6shHLARv-_iI3yRVfLVyoNYWRtaW5pc3RyYXRvcg&amp;cameraId=4xIx1DOxSE00M040NE9LMddLTsw1MBASEE7vKs8odvBuM3dQEV72LA4A&amp;format=fmp4&amp;frames=all&amp;media=video&amp;quality=high&amp;t=live" type="video/fmp4">
        </video> -->
    </div>
    <div class="w-full md:w-1/3 p-2 sm:relative flex flex-col">
        <div class="w-full">
            <p>Chat</p>
        </div>
        <div class="w-full overflow-y-auto" style="max-height: 60vh;" id="list-chats">
            <ul class="flex flex-col gap-2">
                {#each chatMessages as message}
                    <li class="mx-3 bg-gray-100 p-3 rounded-lg">
                        <p class="text-sm text-gray-800">{message}</p>
                    </li>
                {/each}
            </ul>
        </div>
        <div class="fixed md:absolute w-full bottom-0 right-0 p-2 flex gap-2">
            <input
                class="shadow appearance-none border rounded w-full py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:shadow-outline"
                type="text"
                id="chat-message"
                placeholder="Chat..."
                on:submit={submitChat}
                on:keypress={(e) => e.key === "Enter" && submitChat()}
            />
            <button
                class="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded"
                on:click={submitChat}>Send</button
            >
        </div>
    </div>
</section>
