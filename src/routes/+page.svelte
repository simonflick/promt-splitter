<script lang="ts">
	let chunkSize: number = 15000;
	let inputText: string = '';

	$: chunks = getChunks(inputText, chunkSize);

	interface Chunk {
		checked: boolean;
		text: string;
	}

	function getChunks(text: string, chunkSize: number): Chunk[] {
		let chunks: Chunk[] = [];

		while (text.length > 0) {
			chunks.push({ checked: false, text: text.substring(0, chunkSize) });
			text = text.substring(chunkSize, text.length);
		}

		if (chunks.length === 0) {
			return [];
		}

		for (let i = 0; i < chunks.length; i++) {
			chunks[i].text = `Do not answer yet. This is just another part of the text I want to send you. Just receive and acknowledge as "Part ${i + 1}/${chunks.length} received" and wait for the next part.
[START PART ${i + 1}/${chunks.length}]
${chunks[i].text}
[END PART ${i + 1}/${chunks.length}]`;
		}

		chunks.unshift({
			checked: false,
			text: `The total length of the content that I want to send you is too large to send in only one piece. For sending you that content, I will follow this rule:

[START PART 1/10]
this is the content of the part 1 out of 10 in total
[END PART 1/10]
        
Then you just answer: "Received part 1/10". And when I tell you "ALL PARTS SENT", then you can continue processing the data and answering my requests.`
		});

		chunks[chunks.length - 1].text = `${chunks[chunks.length - 1].text}\n
ALL PARTS SENT. Now you can continue processing the request.`;

		return chunks;
	}
</script>

<h1>Prompt Splitter</h1>

<h2>Input</h2>
<label for="chunk-size">Chunk Size</label>
<input id="chunk-size" type="number" min="1000" max="99999999" bind:value={chunkSize} /><br><br>
<textarea cols="100" rows="20" contenteditable bind:value={inputText} />

{#if chunks && chunks.length > 0}
	<h2>{chunks.length - 1} {chunks.length === 2 ? 'Chunk' : 'Chunks'}</h2>

	<ol class="chunks">
		{#each chunks as chunk, i}
			<li class="chunk" class:checked={chunk.checked}>
				<button
					on:click={async () => {
						await navigator.clipboard.writeText(chunk.text);
						chunk.checked = true;
					}}>Copy text</button
				>
				<input id="chunk-{i}" type="checkbox" bind:checked={chunk.checked} />
				<label for="chunk-{i}">
					{chunk.text}
				</label>
			</li>
		{/each}
	</ol>
{/if}

<style>
	.chunks {
		display: flex;
		flex-direction: column;
		gap: 12px;
		list-style: none;
		padding: 0;
	}

	.chunk {
		display: flex;
		gap: 20px;
		border: 1px solid #ddd;
		border-radius: 5px;
		padding: 0 10px;
        align-items: center;
	}

	.chunk.checked {
		background-color: #eee;
	}

	.chunk button {
		flex-shrink: 0;
        height: 2.5rem;
        margin: 10px 0;
	}

    .chunk input {
        width: 20px;
        height: 20px;
        flex-shrink: 0;
    }

	.chunk label {
        font-family: monospace;
        display: -webkit-box;
        -webkit-line-clamp: 5;
        -webkit-box-orient: vertical;
        text-overflow: ellipsis;
        margin: 10px 0;
        word-break: break-all;
        overflow: hidden;
	}
</style>
