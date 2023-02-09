<script lang="ts">
    import { tick } from "svelte";
    import { search } from "./search";
    import { story } from "./story";

    let heldKeys: string[] = [];

    const getCaretIndex = (e: any) => {
        let position = -1;
        const isSupported = typeof window.getSelection !== "undefined";
        if (isSupported) {
            const selection = window.getSelection();
            if (selection !== null && selection.rangeCount !== 0) {
                let range = selection.getRangeAt(0);
                let preCaretRange = range.cloneRange();
                preCaretRange.selectNodeContents(e);
                preCaretRange.setEnd(range.endContainer, range.endOffset);
                position = preCaretRange.toString().length;
            }
        }
        return position;
    }

    const shiftfocus: any = async (from: number, shift: number) => {
        let cells =  [...document.querySelectorAll('.cell')] as HTMLElement[];
        let to = from + shift
        
        if (to < 0 || to >= cells.length) {
            to = Math.min(Math.max(to, 0), cells.length)
            await insertCell(to);
            cells = [...document.querySelectorAll('.cell')] as HTMLElement[];
        }

        cells[to].focus();
        if (shift < 0) {
            let selection = window.getSelection() as Selection;
            selection.selectAllChildren(cells[to]);
            selection.collapseToEnd();
        }
    }

    const insertCell: any = async (i: number) => {
        let cell = {word:"", gloss: "", results: []}
        story.text = [...story.text.slice(0, i), cell, ...story.text.slice(i)]
        await tick();
    }

    const removeCell: any = async (i: number) => {
        story.text = [...story.text.slice(0,i), ...story.text.slice(i+1)]
        await tick();
    }


    let onKeydown: any = async (e: any) => {
        let c = document.activeElement;
        let cells =  [...document.querySelectorAll('.cell')];
        let i = c == null ? 0 : cells.indexOf(c);
        let caretIdx = getCaretIndex(e.target)
        switch (e.key) {
            case " ":
                if (heldKeys.includes("Shift")) {
                    break;
                }
                e.preventDefault();
                await insertCell(i+1)
                await shiftfocus(i,+1)
                story.text[i].word = cells[i].innerHTML.slice(0, caretIdx);
                story.text[i+1].word = cells[i].innerHTML.slice(caretIdx);
                break;
            case "ArrowLeft":
                if (caretIdx == 0) {
                    e.preventDefault();
                    shiftfocus(i, -1)
                }
                break;
            case "ArrowRight":
                if (caretIdx == e.target.innerHTML.length) {
                    e.preventDefault();
                    shiftfocus(i, +1)
                }
                break;
            case "Backspace":
                if (caretIdx == 0) {
                    if (e.target.innerHTML.length === 0) {
                        removeCell(i);
                        e.preventDefault();
                    }
                    if (i > 0) {
                        shiftfocus(i, -1);
                    }
                }
                break;
        }
    }

    let plainPaste = (e: ClipboardEvent) => {
        let clipboard = e.clipboardData?.getData('text/plain') || "";
        document.execCommand('inserttext', false, clipboard);
    }

    // $: idxs = story.text.reduce((acc, t, i) => (!t.word.endsWith("-") ? [...acc, i+1] : acc), [])
    $: chunks = story.text.reduce(
        (acc, t, i) => !t.word.endsWith("-") ? [...acc, i+1] : acc,
        [0]
    );
</script>

<header class="border-b border-black pb-3 mx-auto text-center">
    <h1 class="font-bold text-xl">{story.title.toUpperCase()}</h1>
    <h1 class="font-bold text-xl mb-4">{story.subtitle.toUpperCase()}</h1>
    <h2 class="italic">{story.editor}</h2>
</header>

<main class="pt-10">
    <h2 class="text-lg font-bold">Speaker: {story.speaker}</h2>
    <div class="flex flex-wrap gap-y-3">
        {#each {length: chunks.length - 1} as _, i}
            {@const a = chunks[i]}
            {@const b = chunks[i+1]}
            {@const chunk = story.text.slice(a, b)}
            <div>
                <div class="flex">
                    {#each chunk as t, i}
                        <span
                            contenteditable=true
                            spellcheck=false
                            bind:innerHTML={t.word}
                            on:keydown={onKeydown}
                            on:paste|preventDefault={plainPaste}
                            class="
                                cell
                                italic
                                outline-none
                                py-0.5
                                empty:before:content-['…'] empty:text-gray-300
                                hover:bg-gray-100
                                focus:bg-gray-200
                                {heldKeys.includes('Shift') ? 'bg-red-50' : ''}
                                {t.word.endsWith('-') ? '' : 'w-full pr-3'}
                            "
                        >{t.word}</span>
                    {/each}
                </div>
                <div class="pr-3">
                    {#each chunk as t}
                        <span
                            class="
                                text-sm
                                {t.word.endsWith('-') ? '' : 'w-full pr-3'}
                            "
                        >{t.gloss}{t.word.endsWith('-') ? '-' : ''}</span>
                    {/each}
                </div>
            </div>
        {/each}
    </div>
</main>

<svelte:window
    on:keydown={e => heldKeys = [e.key, ...heldKeys]}
    on:keyup={e => heldKeys = heldKeys.filter(x => x !== e.key)}
/>