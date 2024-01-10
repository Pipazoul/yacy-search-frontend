<script lang="ts">
	import { onMount } from 'svelte';

	interface SearchResults {
		channels: [
			{
				title: string;
				description: string;
				link: string;
				image: {
					url: string;
					title: string;
					link: string;
				};
				startIndex: string;
				itemsPerPage: string;
				searchTerms: string;
				totalResults: string;
				items: [
					{
						title: string;
						link: string;
						code: string;
						description: string;
						pubDate: string;
						image: string;
						size: string;
						sizename: string;
						guid: string;
						faviconUrl: string;
						host: string;
						path: string;
						urlhash: string;
						ranking: string;
					}
				];
				navigation: [
					{
						displayName: string;
						facetname: 'protocols' | 'hosts' | 'authors' | 'filetypes' | 'languages' | 'topics';
						max: string;
						mean: string;
						min: string;
						type: string;
						elements: [
							{
								name: string;
								count: string;
								modifier: string;
								url: string;
							}
						];
					}
				];
			}
		];
	}

	const yacyUrl = 'https://search.doesnotexist.club';
	let request = 'lyon';
	let lastRequest = '';
	let results = {} as SearchResults;

    let currentSearch = 'Images';

	onMount(async () => {
		console.log('mounted');
		setInterval(async () => {
			//console.log('request', request);
			if (request !== lastRequest) {
                await fetchResults();
			}
		}, 2000);
	});


    function setRequestType(type: string) {
        currentSearch = type;
        fetchResults();
    }

    async function fetchResults() {
        let response;
        switch (currentSearch) {
                    case "Documents":
                        console.log("Documents");
                        lastRequest = request;
                        response = await fetch(yacyUrl + "/yacysearch.json?query=" + request + "/date &maximumRecords=400");
                        results = await response.json();
                        break;
                    case "Images":
                        console.log("Images");
                        lastRequest = request;
                        response = await fetch(yacyUrl + "/yacysearch.json?query=" + request + "/date &maximumRecords=400&contentdom=image");
                        results = await response.json();
                        break;
                    case "Actu":
                        console.log("Actu");
                        break;
                    default:
                        break;
        }
    }

    function htmlDecode(url: string) {
        //remove amp;
        let doc = new DOMParser().parseFromString(url, "text/html");
        return doc.documentElement.textContent;
    }

    function trimString(str: string) {
        return htmlDecode(str).substring(0, 200) + " [...]";
    }

</script>

<section>
	<div class="p-4 sticky top-0 bg-white">
        <div class="flex items-center justify-center border border-black rounded-lg">
            <svg xmlns="http://www.w3.org/2000/svg" height="16" width="16" viewBox="0 0 512 512"><!--!Font Awesome Free 6.5.1 by @fontawesome - https://fontawesome.com License - https://fontawesome.com/license/free Copyright 2024 Fonticons, Inc.--><path d="M416 208c0 45.9-14.9 88.3-40 122.7L502.6 457.4c12.5 12.5 12.5 32.8 0 45.3s-32.8 12.5-45.3 0L330.7 376c-34.4 25.2-76.8 40-122.7 40C93.1 416 0 322.9 0 208S93.1 0 208 0S416 93.1 416 208zM208 352a144 144 0 1 0 0-288 144 144 0 1 0 0 288z"/></svg>
            <input
                type="text"
                placeholder="Type here"
                class="input  w-full max-w-xs"
                bind:value={request}
            />
        </div>
        <div class="mt-2">
            <button class="rounded-full border border-1 border-black pt-2 pb-2 pl-3 pr-3 {currentSearch == "Documents" ? "bg-black text-white" : ""} " on:click={() => setRequestType("Documents")}>Documents</button>
            <button class="rounded-full border border-1 border-black pt-2 pb-2 pl-3 pr-3 {currentSearch == "Images" ? "bg-black text-white" : ""} " on:click={() => setRequestType("Images")}>Images</button>
            <button class="rounded-full border border-1 border-black pt-2 pb-2 pl-3 pr-3 {currentSearch == "Actu" ? "bg-black text-white" : ""} " on:click={() => setRequestType("Actu")}>Actu</button>
        </div>
	</div>
   
	<div>
		{#if results.channels && currentSearch == "Documents"}
			{#each results.channels as channel}
				<div >
					<ul class="flex flex-wrap"> 
                            {#each channel.items as item}
                            
                                <li class="">
                                    <a href="{item.link}" target="_blank">
                                    <div class="flex items-center p-4">
                                        {#if item.faviconUrl}
                                        <div class="bg-slate-200 rounded-full w-6 h-6 mr-3">
                                            <img src={htmlDecode(yacyUrl + "/" + item.faviconUrl)}  />
                                        </div>
                                        {/if}
                                        <p>{item.title}</p>
                                    </div>
                                    
                                    {#if item.image}
                                        <div class="flex p-4">
                                            <div class="w-1/2 h-40 bg-cover bg-center rounded-md" style="background-image: url({item.image});"></div>
                                            <p class="w-1/2 text-sm ml-2">{trimString(item.description)}</p>
                                        </div>
                                        {:else}
                                        <p>{trimString(item.description)}</p>
                                    {/if}
                                </a>
                                </li>
                            {/each}
					</ul>
				</div>
			{/each}
		{/if}
        {#if results.channels && currentSearch == "Images"}
			{#each results.channels as channel}
				<div>
					<ul class="flex flex-wrap"> 
                            {#each channel.items as item}
                            <a class="w-1/2" href="{item.url}" target="_blank">
                                <li>
                                    <div class="flex items-center p-4">
                                        <p>{item.title}</p>
                                    </div>
                                    {#if item.image}
                                        <div class="flex p-4">
                                            <div class="bg-cover w-full h-40 bg-center rounded-md" style="background-image: url({item.image});"></div>
                                        </div>
                                    {/if}

                                </li>
                            </a>
                            {/each}
					</ul>
				</div>
			{/each}
		{/if}
	</div>
</section>
