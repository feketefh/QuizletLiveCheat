<script lang="ts">
    import { onMount, onDestroy, createEventDispatcher } from "svelte";

    // Event callback props
    export let onanswer: (() => void) | undefined = undefined;
    export let onhelpMode: ((event: CustomEvent<number>) => void) | undefined = undefined;

    let visible = true;
    const helpModes = ['None', 'Auto Answer (instant)', 'Auto Answer (wait)', 'Outline Correct Answer']
    let helpMode = 0;
    let lastHelpMode = 0;
    let smallDevice: boolean = false;

    let tapCount = 0;
    let lastTapTime = 0;
    let tapTimeout: ReturnType<typeof setTimeout> | null = null;
    
    const TAP_DELAY = 300; // ms between taps
    const RESET_DELAY = 400; // ms to reset tap count
    
    const attachListener = (node: any): void => {
      // attach a media query listener to the window
      const mediaQuery: MediaQueryList = window.matchMedia('(width <= 640px)');

      // every time the media query matches or unmatches
      mediaQuery.addEventListener('change', ({ matches }: MediaQueryListEvent) => {
        // set the state of our variable
        smallDevice = matches;
      });
    }

    function handleTouchEnd(e: TouchEvent) {
        const now = Date.now();
        const timeSinceLastTap = now - lastTapTime;

        // Clear existing timeout
        if (tapTimeout) {
            clearTimeout(tapTimeout);
        }

        // Increment or reset tap count based on timing
        if (timeSinceLastTap < TAP_DELAY) {
            tapCount++;
        } else {
            tapCount = 1;
        }

        lastTapTime = now;

        // Check if we've reached triple tap
        if (tapCount === 3) {
            multiTouchHandler();
            tapCount = 0;
            lastTapTime = 0;
        } else {
            // Reset tap count after delay if no more taps
            tapTimeout = setTimeout(() => {
                tapCount = 0;
            }, RESET_DELAY);
        }
    }

    onMount(() => {
        document.addEventListener('touchend', handleTouchEnd);
    });

    onDestroy(() => {
        document.removeEventListener('touchend', handleTouchEnd);
        if (tapTimeout) {
            clearTimeout(tapTimeout);
        }
    });

    let dispatch = createEventDispatcher();

    function changeHelpMode(change: number) {
        helpMode += change;
        if (helpMode < 0) helpMode += helpModes.length;
        helpMode %= helpModes.length;

        // Call both dispatcher and callback prop
        dispatch('helpMode', helpMode);
        onhelpMode?.(new CustomEvent('helpMode', { detail: helpMode }));
    }

    function close() {
        visible = false;
        helpMode = 0;

        dispatch('helpMode', helpMode);
        onhelpMode?.(new CustomEvent('helpMode', { detail: helpMode }));
    }

    function handleAnswer() {
        dispatch('answer');
        onanswer?.();
    }

    function onKeyDown(event: KeyboardEvent) {
        if(event.key !== '\\') return;

        if (visible == true) {
            lastHelpMode = helpMode;
            helpMode = 0;
        }
        else {
            helpMode = lastHelpMode;
        }

        dispatch('helpMode', helpMode);
        visible = !visible;
    }

    function multiTouchHandler() {
        if (visible == true) {
            lastHelpMode = helpMode;
            helpMode = 0;
        }
        else {
            helpMode = lastHelpMode;
        }

        dispatch('helpMode', helpMode);
        visible = !visible;
    }
</script>

<svelte:window on:keydown={onKeyDown} use:attachListener/>


<section>
    {#if visible}
        <div class="hud">
            <button on:click={handleAnswer} class="answer">Answer Question</button>
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
</section>

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