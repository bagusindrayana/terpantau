<script>
    /**
     * Represents a item with name and stream attribute.
     * @typedef {Object} Item
     */
    export let item;

    
</script>

<div class="lg:w-1/3 sm:w-1/2 p-4">
    <a class="flex relative overflow-hidden" href="/stream?stream={btoa(item.stream)}">
        <!-- <img
            alt="gallery"
            class="absolute inset-0 w-full h-full object-cover object-center"
            src="https://dummyimage.com/600x360"
        /> -->
        <div class="absolute inset-0 w-full h-full object-cover object-center bg-gray-300">
            <video
            
            crossorigin
            
            class=" w-full h-full "
        >
            <source type="application/x-mpegURL" src={item.stream} />
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
