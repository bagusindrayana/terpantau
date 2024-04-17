<script>
    import { onMount } from "svelte";
    /**
     * Represents a item with name and stream attribute.
     * @type {Object}
     */
    export let item;
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
            method: "HEAD"
        }).then((response) => {
            if (response.headers.get("content-type") === "application/octet-stream") {
                isStream = true;
                initVideo();
            } else {
                isStream = false;
            }
        });
    }


    const initVideo = () => {
        const video = document.querySelector("#video-"+id);
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
    }

    onMount(() => {
        console.log(item.stream);
        checkSteam(item.stream);
        
    });
</script>

<div class="lg:w-1/3 sm:w-1/2 p-4">
    <a
        class="flex relative overflow-hidden"
        href="/stream?stream={btoa(JSON.stringify(item))}"
    >
        <!-- <img
            alt="gallery"
            class="absolute inset-0 w-full h-full object-cover object-center"
            src="https://dummyimage.com/600x360"
        /> -->
        <div
            class="absolute inset-0 w-full h-full object-cover object-center bg-gray-300"
        >
            <video id={"video-"+id} crossorigin class=" w-full h-full">
                <source src={item.stream} />
            </video>
        </div>

        <div
            class="px-8 py-10 relative z-10 w-full border-4 border-gray-200 bg-white opacity-0 hover:opacity-100"
        >
            <h2
                class="tracking-widest text-sm title-font font-medium text-indigo-500 mb-1"
            >
                {item.name}
            </h2>
            <h1 class="title-font text-lg font-medium text-gray-900 mb-3">
                Shooting Stars
            </h1>
            <p class="leading-relaxed">
                Photo booth fam kinfolk cold-pressed sriracha leggings jianbing
                microdosing tousled waistcoat.
            </p>
        </div>
    </a>
</div>
