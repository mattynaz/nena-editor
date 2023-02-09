<script lang="ts">
    let story = [
        {
            word: "laxma",
            gloss: "1",
        },
        {
            word: "laxma",
            gloss: "1",
        },
        {
            word: "laxma",
            gloss: "1",
        },
        {
            word: "laxma",
            gloss: "1",
        },
        {
            word: "laxma",
            gloss: "1",
        },
        {
            word: "laxma",
            gloss: "1",
        },
        {
            word: "laxma",
            gloss: "1",
        },
        {
            word: "laxma",
            gloss: "1",
        },
        {
            word: "laxma",
            gloss: "1",
        },
        {
            word: "laxma",
            gloss: "1",
        },
        {
            word: "laxma",
            gloss: "1",
        },
        {
            word: "laxma",
            gloss: "1",
        },
        {
            word: "laxma",
            gloss: "1",
        },
        {
            word: "laxma",
            gloss: "1",
        },
        {
            word: "laxma",
            gloss: "1",
        },
    ]
</script>

<div class="border w-full h-full flex gap-1 flex-wrap">
    {#each story as s}
        <div>
            <p contenteditable="">{s.word}</p>
            <p class="select-none">{s.gloss}</p>
        </div>
    {/each}
</div>
