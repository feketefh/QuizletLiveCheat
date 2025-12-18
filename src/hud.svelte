<script lang="ts">
    import { onMount, onDestroy, createEventDispatcher } from "svelte";

    // Event callback props
    export let onanswer: (() => void) | undefined = undefined;
    export let onhelpMode: ((event: CustomEvent<number>) => void) | undefined = undefined;

    let visible = true;
    const helpModes = ['None', 'Auto Answer (instant)', 'Auto Answer (wait)', 'Outline Correct Answer']
    let helpMode = 0;
    let lastHelpMode = 0;

    let tapCount = 0;
    let lastTapTime = 0;
    let tapTimeout: ReturnType<typeof setTimeout> | null = null;
    
    let x = 10;
    let y = 10;
    let isDragging: boolean = false;
    let hasMoved: boolean = false;
    let startX: number;
    let startY: number;

    const TAP_DELAY = 300;
    const RESET_DELAY = 400;

    function handleTouchEnd(e: TouchEvent) {
        // Don't count as tap if we were dragging
        if (hasMoved) {
            return;
        }

        const now = Date.now();
        const timeSinceLastTap = now - lastTapTime;

        if (tapTimeout) {
            clearTimeout(tapTimeout);
        }

        if (timeSinceLastTap < TAP_DELAY) {
            tapCount++;
        } else {
            tapCount = 1;
        }

        lastTapTime = now;

        if (tapCount === 3) {
            multiTouchHandler();
            tapCount = 0;
            lastTapTime = 0;
        } else {
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

    function handleStart(event: MouseEvent | TouchEvent) {
        isDragging = true;
        hasMoved = false;

        if (event.type === 'touchstart') {
          event.preventDefault();
        }

        const clientX = event.type === 'mousedown' ? (event as MouseEvent).clientX : (event as TouchEvent).touches[0].clientX;
        const clientY = event.type === 'mousedown' ? (event as MouseEvent).clientY : (event as TouchEvent).touches[0].clientY;

        startX = clientX - x;
        startY = clientY - y;
    }

    function handleMove(event: MouseEvent | TouchEvent) {
        if (!isDragging) return;

        hasMoved = true;
        event.preventDefault();

        const clientX = event.type === 'mousemove' ? (event as MouseEvent).clientX : (event as TouchEvent).touches[0].clientX;
        const clientY = event.type === 'mousemove' ? (event as MouseEvent).clientY : (event as TouchEvent).touches[0].clientY;

        requestAnimationFrame(() => {
            x = clientX - startX;
            y = clientY - startY;
        });
    }

    function handleEnd() {
      isDragging = false;
      // Reset hasMoved after a short delay to allow handleTouchEnd to check it
      setTimeout(() => {
        hasMoved = false;
      }, 50);
    }

    let dispatch = createEventDispatcher();

    function changeHelpMode(change: number) {
        helpMode += change;
        if (helpMode < 0) helpMode += helpModes.length;
        helpMode %= helpModes.length;

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

<svelte:window 
    on:keydown={onKeyDown} 
    on:mousemove={handleMove}
    on:mouseup={handleEnd}
    on:touchmove={handleMove}
    on:touchend={handleEnd}
/>


<section>
    {#if visible}
        <div class="hud" 
            style="transform: translate({x}px, {y}px);" 
            on:mousedown={handleStart} 
            on:touchstart|preventDefault={handleStart}
            role="button"
            tabindex="0"
        >
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
    will-change: transform;
    touch-action: none;
    user-select: none;
    cursor: move;
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