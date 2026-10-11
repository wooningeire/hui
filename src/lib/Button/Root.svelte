<script lang="ts">
import type { Snippet } from "svelte";
import type { HTMLButtonAttributes } from "svelte/elements";

let {
    children,
    keyHeld = false,
    displayClass,
    buttonStyle = "text",
    noBg = false,
    ...buttonProps
}: {
    children: Snippet,
    keyHeld?: boolean,
    buttonStyle?: "text" | "image" | "icon",
    displayClass?: string,
    noBg?: boolean,
} & HTMLButtonAttributes = $props();
</script>

<button
    class:no-bg={noBg}
    {...buttonProps}
>
    <button-display
        class:key-held={keyHeld}
        class:text-button={buttonStyle === "text"}
        class:icon-button={buttonStyle === "icon"}
        class={displayClass}
        class:no-bg={noBg}
    >
        {@render children?.()}
    </button-display>
</button>

<style lang="scss">
$bg-col: var(--bg-col, oklch(1 0 0));

button {
    margin: 0;
    padding: 0;
    display: grid;
    place-items: stretch;

    border: none;
    background: none;


    font-family: var(--font-family);

    --bg-stripe-1-col: oklch(from #{$bg-col} l c h / 0.125);
    --bg-stripe-2-col: oklch(from #{$bg-col} l c h / 0.25);
    --inset-box-shadow: 0 1em 2em oklch(from #{$bg-col} l c h / 0.0625) inset;
    --outset-box-shadow: 0 0.25rem 1rem 0.5rem oklch(0 0 0 / 0.125);

    
    &:not(.no-bg) {
        cursor: pointer;
    }
    
    &[disabled] {
        pointer-events: none;
        opacity: 0.3;
    }

    button-display {
        display: block;
        // border-radius: 1rem;

        pointer-events: none;
        overflow: hidden;

        transition:
            transform 0.25s cubic-bezier(0,2.75,.47,1),
            box-shadow 0.1s cubic-bezier(0,1,.47,1),
            filter 0.25s ease;

        &.text-button {
            padding: 0.25rem 1rem;
        }

        &.icon-button {
            padding: 0.5rem;
        }

        &:not(.no-bg) {
            background: var(--bg-stripe-1-col);
            // box-shadow:
            //     $outset-box-shadow,
            //     $inset-box-shadow;

            backdrop-filter: blur(4px);
        }
    }

    &:is(:hover, :focus-visible) > button-display:not(.no-bg),
    button-display.key-held:not(.no-bg) {
        transform: translateY(-0.125rem) scale(1.05);

        animation: sliding-background 1s infinite linear;
        background-color: var(--bg-stripe-1-col);
        background-image: repeating-linear-gradient(
            135deg,
            var(--bg-stripe-1-col) 0,
            var(--bg-stripe-1-col) 1rem,
            var(--bg-stripe-2-col) 1rem,
            var(--bg-stripe-2-col) 2rem,
        );
        background-size: calc(100% + 3rem) calc(100% + 3rem);

        z-index: 1;

        @keyframes sliding-background {
            from {
                background-position: -2.828427rem -2.828427rem; // 2 * sqrt(2)
            }
            to {
                background-position: 0 0;
            }
        }
    }
}

:global(a:active) button-display:not(.no-bg),
button:active > button-display:not(.no-bg),
button-display.key-held:not(.no-bg) {
    transform: translateY(0.0625rem) scale(0.95);
}
</style>