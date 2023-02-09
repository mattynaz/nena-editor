<script lang="ts">
    import { tick } from "svelte";
    import { search } from "../search";

    let shiftPressed: boolean = false;

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

    let onKeypress: any = (e: any) => {
        // alert("keypress", e.key)
    }

    let onKeydown: any = (e: any) => {
        let c = document.activeElement;
        let cells =  [...document.querySelectorAll('.cell')];
        let i = c == null ? 0 : cells.indexOf(c);

        switch (e.key) {
            case "ArrowLeft":
                if (getCaretIndex(e.target) == 0) {
                    e.preventDefault();
                    shiftfocus(i, -1);
                }
                break;
            case "ArrowRight":
                if (getCaretIndex(e.target) == e.target.innerHTML.length) {
                    e.preventDefault();
                    shiftfocus(i, +1)
                }
                break;
            case "Backspace":
                if (getCaretIndex(e.target) == 0) {
                    if (e.target.innerHTML.length === 0) {
                        removeCell(i);
                        e.preventDefault();
                    }
                    shiftfocus(i, -1);
                }
                break;
            case "Tab":
                shiftfocus(i, +1)
                e.preventDefault();
                break;
        }
    }

    let onKeyup: any = (e: any) => {
        // alert("keypress", e.key)
    }


    let onInput: any = (e: any) => {
        // console.log("input", e.target.innerHTML)
        // console.log(search(e.target.innerHTML))
    }

    let onPaste = (e: ClipboardEvent) => {
        let clipboard = e.clipboardData?.getData('text/plain') || "";
        document.execCommand('inserttext', false, clipboard);
    }

    const insertCell: any = async (i: number) => {
        if (i === -1) {
            i = story.text.length;
        }
        if (story.text[i-1]?.word.length > 0 && story.text[i]?.word.length > 0) {
            let cell = {word:"", gloss: "", results: []}
            story.text = [...story.text.slice(0,i), cell, ...story.text.slice(i)]
            await tick();
        }
        let items =  [...document.querySelectorAll('.cell')];
        items[i].focus();
    }

    const removeCell: any = async (i: number) => {
        story.text = [...story.text.slice(0,i), ...story.text.slice(i+1)]
        await tick();
    }

    const shiftfocus: any = async (i: number, shift: number) => {
        let cells =  [...document.querySelectorAll('.cell')];
        let j = i+shift
        if (j === cells.length) {
            await insertCell(j);
            cells = [...document.querySelectorAll('.cell')];
        }

        j = Math.min(Math.max(0, j), cells.length-1)
        cells[j].focus();
        if (shift < 0) {
            let selection = window.getSelection() as Selection;
            selection.selectAllChildren(cells[j]);
            selection.collapseToEnd();
        }
    }

    // const chunkify = (text: any) => {
    //     let idxs = text.reduce((acc, t, i) => (!t.word.endsWith("-") ? [...acc, i+1] : acc), [])
    //     let chunks = [];
    //     idxs.forEach((end, i) => {
    //         let start = i === 0 ? 0 : idxs[i-1];
    //         chunks.push(text.slice(start, end));
    //     });
    //     return chunks;
    // }

    let story = {
        title: "Matthew In His Dorm",
        subtitle: "Text 1: §1-9",
        speaker: "Matthew Nazari",
        editor: "Matthew Nazari",
        text: [
            {
                "word": "xa-",
                "gloss": "one",
                "results": [],
            },
            {
                "word": "yúma",
                "gloss": "day",
                "results": [],
            },
            {
                "word": "ʾə́tva",
                "gloss": "there_is-pst",
                "results": [],
            },
            {
                "word": "xá",
                "gloss": "a",
                "results": [],
            },
            {
                "word": "⁺málla.",
                "gloss": "Mullah",
                "results": [],
            },
            {
                "word": "ʾa-",
                "gloss": "this",
                "results": [],
            },
            {
                "word": "⁺málla",
                "gloss": "Mullah",
                "results": [],
            },
            {
                "word": "bərrə́xšələ",
                "gloss": "to_go.prog-cop.prs.3ms",
                "results": [],
            },
            {
                "word": "⁺ʾál",
                "gloss": "to",
                "results": [],
            },
            {
                "word": "⁺tájər",
                "gloss": "merchant",
                "results": [],
            },
        ],
    }

    $: idxs = story.text.reduce((acc, t, i) => (!t.word.endsWith("-") ? [...acc, i+1] : acc), [])

</script>

<header class="border-b border-black pb-3 mx-auto text-center">
    <h1 class="font-bold text-xl">{story.title.toUpperCase()}</h1>
    <h1 class="font-bold text-xl mb-4">{story.subtitle.toUpperCase()}</h1>
    <h2 class="italic">{story.editor}</h2>
</header>

<main class="pt-10">
    <h2 class="text-lg font-bold">Speaker: {story.speaker}</h2>

    <!-- <div class="italic">
        {#each story.text as t}
            {t.word}{t.word.endsWith('-') ? '' : ' '}
        {/each}
    </div> -->

    <div class="flex flex-wrap gap-y-3 story -ml-1">
        {#each idxs as _, i}
        {@const chunk = story.text.slice(idxs[i-1] ||0, idxs[i])}
            <div>
                <div class="flex">
                    {#each chunk as t, j}
                        <div class="relative {!t.word.endsWith("-") ? 'w-full' : ''}">
                            <!-- svelte-ignore a11y-click-events-have-key-events -->
                            <span
                                on:click={()=>{
                                    let k = (idxs[i-1] || 0) + j
                                    if (!shiftPressed) {insertCell(k + 1)}
                                    else {removeCell(k)}
                                }}
                                class="
                                    peer
                                    select-none
                                    absolute
                                    -top-1
                                    -right-1
                                    font-bold
                                    z-10
                                    cursor-pointer
                                    text-green-600
                                    leading-none
                                    text-xs
                                    animate-bounce
                                    opacity-0
                                    hover:opacity-100
                                "
                            >
                                {shiftPressed ? '❌' : '⬇'}
                            </span>
                            <p
                                contenteditable=true
                                spellcheck=false
                                on:keydown={onKeydown}
                                on:keyup={onKeyup}
                                on:keypress={onKeypress}
                                on:input={onInput}
                                class="
                                    cell
                                    empty:before:content-['…']
                                    empty:text-gray-300
                                    italic
                                    hover:bg-gray-100
                                    {shiftPressed ? 'peer-hover:bg-red-50' : ''}
                                    focus:bg-gray-200
                                    outline-none
                                    py-0.5
                                    {j === 0 ? 'pl-1' : ''}
                                    {t.word.endsWith('-') ? '' : 'pr-1'}
                                "
                            >{t.word}</p>
                        </div>
                    {/each}
                </div>
                <div class="flex">
                    {#each chunk as t, j}
                        <p class="
                            text-sm
                            select-none
                            {j === 0 ? 'pl-1' : ''}
                            {t.word.endsWith('-') ? '' : 'pr-1'}
                        ">{t.gloss}{t.word.endsWith('-') ? '-' : ''}</p>
                    {/each}                    
                </div>
            </div>
        {/each}
    </div>

</main>

<svelte:window
    on:keydown={(e)=>{if (e.key === "Shift") shiftPressed = true}}
    on:keyup={(e)=>{if (e.key === "Shift") shiftPressed = false}}
/>