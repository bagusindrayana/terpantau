<script>
    import { page } from "$app/stores";
    import { onMount } from "svelte";
    import Plyr from "plyr";
    import Hls from "hls.js";
    import { PUBLIC_API_CORS, PUBLIC_API_LIVECHAT } from "$env/static/public";
    import Player from "../../../components/Player.svelte";
    import Pusher from "pusher-js";

    var streamInfo = JSON.parse(atob($page.url.searchParams.get("stream")));


    /**
     * Represents a chat message.
     * @type {Array<Object>}
     */
    let chatMessages = [];

    

    //generate channel name based streamInfo.stream, remove http and symbol

    /**
     *
     * @param e {Event}
     */
    function submitChat() {
        const cm = document.getElementById("chat-message");
        if (cm) {
            const url = PUBLIC_API_LIVECHAT + "/send-message";
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
<section
    class="w-full flex flex-col md:flex-row gap-3"
    style="height: 70vh;"
>
    <div class="w-full h-full relative md:w-2/3 p-2 bg-black flex justify-center items-center  video-stream-detail">
        <!-- {#if streamInfo != undefined && streamInfo != null}
            <video
                controls
                crossorigin
                playsinline
                class="w-full  "
            >
                <source src={streamLink} />
            </video>
        {:else}
            <p>Loading...</p>
        {/if} -->
        <!-- <video name="media" class="stream__frame" id="video_stream" autoplay muted playsinline poster="https://vmssamarinda.iconpln.co.id:8443/mt/api/rest/v1/media?session=AYMAfP8KEOFwg8Ds1UExjWk1VvcCCekSFgoU5v9_dWIirrX3qCM7NVtkQOoF1zQaEAK1f5vl_EV7ssdxdYw0vdMiEG4SCpxUgk9ivN4Jozv2qgkqEFBlbWtvdCBTYW1hcmluZGEyIRoQTZ6shHLARv-_iI3yRVfLVyoNYWRtaW5pc3RyYXRvcg&amp;cameraId=4xIx1DOxSE00M040NE9LMddLTsw1MBASEE7vKs8odvBuM3dQEV72LA4A&amp;format=jpeg&amp;frames=all&amp;media=video&amp;quality=high&amp;t=live" src="https://vmssamarinda.iconpln.co.id:8443/mt/api/rest/v1/media?session=AYMAfP8KEOFwg8Ds1UExjWk1VvcCCekSFgoU5v9_dWIirrX3qCM7NVtkQOoF1zQaEAK1f5vl_EV7ssdxdYw0vdMiEG4SCpxUgk9ivN4Jozv2qgkqEFBlbWtvdCBTYW1hcmluZGEyIRoQTZ6shHLARv-_iI3yRVfLVyoNYWRtaW5pc3RyYXRvcg&amp;cameraId=4xIx1DOxSE00M040NE9LMddLTsw1MBASEE7vKs8odvBuM3dQEV72LA4A&amp;format=fmp4&amp;frames=all&amp;media=video&amp;quality=high&amp;t=live" type="video/fmp4">
        </video> -->
        <Player streamInfo={streamInfo}  preview={true} id={"stream"}/>
        
    </div>
    <div class="w-full md:w-1/3 p-2 sm:relative flex flex-col">
        <div class="w-full">
            <p>Chat</p>
        </div>
        <div
            class="w-full overflow-y-auto"
            style="max-height: 60vh;"
            id="list-chats"
        >
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
