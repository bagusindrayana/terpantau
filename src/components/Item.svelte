<script>
    import { onMount, onDestroy } from "svelte";
    import { PUBLIC_API_CORS,PUBLIC_API_THUMBNAIL } from "$env/static/public";
    // import Player from "./Player.svelte";
    /**
     * Represents a item with name and stream attribute.
     * @type {Object}
     */
    export let item;
    

    /**
     * Represents a item with name and stream attribute.
     * @type {string|null}
     */
    export let source;

    /**
     * Represents a item with name and stream attribute.
     * @type {string|null}
     */
     let streamLink = item.stream;

   

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
        
    });

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
        <!-- <video id={"video-" + id} crossorigin class=" w-full h-full">
            <source src={streamLink} />
        </video> -->
        <!-- <Player streamInfo={item}  preview={true} id={id}/> -->
        <img src="{PUBLIC_API_THUMBNAIL+encodeURIComponent(item.stream)}" onerror="this.src='/cctv-placeholder.png'" class="h-full"/>
    </div>
    <div class="w-full p-4">
        <div class="px-6 py-4">
            <div class="font-bold text-xl mb-2">{item.name}</div>
            <p class="leading-relaxed">
                Sumber : {source}
            </p>
        </div>
        <!-- <div class="px-6 pt-4 pb-2">
            {#if online}
                <span class="text-green-500">Online</span>
            {:else}
                <span class="text-red-500">Offline</span>
            {/if}
        </div> -->
    </div>
</a>
