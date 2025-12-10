<script lang="ts">
    import { createEventDispatcher } from "svelte";

    let visible = true;
    const helpModes = ['None', 'Auto Answer (instant)', 'Auto Answer (wait)', 'Outline Correct Answer']
    let helpMode = 0;

    let dispatch = createEventDispatcher();

    function changeHelpMode(change: number) {
        helpMode += change;
        if (helpMode < 0) helpMode += helpModes.length;
        helpMode %= helpModes.length;

        dispatch('helpMode', helpMode);
    }

    function onKeyDown(event: KeyboardEvent) {
        if(event.key !== '\\') return;

        visible = !visible;
    }

    function close() {
        visible = false;
        helpMode = 0;

        dispatch('helpMode', helpMode);
    }
</script>

<svelte:window on:keydown={onKeyDown} />

{#if visible}
    <div class="hud">
        <button on:click={() => close()} class="close" aria-label="Close">
            <svg width="14" height="14" viewBox="0 0 14 14" fill="none" xmlns="http://www.w3.org/2000/svg">
                <path d="M1 1L13 13M13 1L1 13" stroke="currentColor" stroke-width="2" stroke-linecap="round"/>
            </svg>
        </button>
        <button on:click={() => dispatch('answer')} class="answer">Answer Question</button>
        <div class="help">
            <div>
                Help Mode
            </div>
            <div class="row helpControl">
                <button on:click={() => changeHelpMode(-1)}>&lt;</button>
                <div class="display">{ helpModes[helpMode] }</div>
                <button on:click={() => changeHelpMode(1)}>&gt;</button>
            </div>
        </div>
    </div>
{/if}

<style>
.hud {
    position: absolute;
    top: 10px;
    left: 10px;
    width: 300px;
    height: 200px;
    z-index: 999999999999;
    background-color: rgba(0, 0, 0, 0.9);
    border-radius: 0.5em;
    display: flex;
    flex-direction: column;
    justify-content: space-evenly;
    align-items: center;
    color: white;
}

.hud .row {
    display: flex;
    flex-direction: row;
    justify-content: space-between;
    align-items: space-between;
    width: 100%;
}

.hud .close {
    position: absolute;
    top: 8px;
    right: 8px;
    width: 28px;
    height: 28px;
    border-radius: 50%;
    background-color: rgba(255, 255, 255, 0.1);
    border: none;
    cursor: pointer;
    display: flex;
    align-items: center;
    justify-content: center;
    color: rgba(255, 255, 255, 0.7);
    transition: all 0.2s ease;
    padding: 0;
}

.hud .close:hover {
    background-color: rgba(255, 255, 255, 0.2);
    color: white;
    transform: rotate(90deg);
}

.hud .close:active {
    transform: rotate(90deg) scale(0.9);
    background-color: rgba(255, 255, 255, 0.15);
}

.hud .answer {
    width: 70%;
    height: 50px;
    font-family: Verdana, Geneva, Tahoma, sans-serif;
    font-size: 1em;
    border-radius: 0.5em;
    background-color: white;
    color: black;
    border: none;
    transition: transform 0.3s ease;
}

.hud .answer:active {
    transform: scale(0.93);
}

.hud .help {
    display: flex;
    flex-direction: column;
    align-items: center;
    width: 85%;
}

.hud .helpControl button {
    width: 25px;
    height: 25px;
    border-radius: 0.5em;
    background-color: white;
    border: none;
    transition: transform 0.3s ease;
    margin: 5px;
    color: black;
}
</style>