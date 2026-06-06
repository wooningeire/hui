<script lang="ts">
    import { Hotkey } from "$lib/index.js";

let {
    label,
}: {
    label: string,
} = $props();

let pressed = $state(false);
</script>

<Hotkey
    key={label}
    onKeyDown={() => pressed = true}
    onKeyUp={() => pressed = false}
/>

<keyboard-key>
    <keyboard-key-underlay></keyboard-key-underlay>

    <kbd class:pressed>{label}</kbd>
</keyboard-key>

<style lang="scss">
keyboard-key {
    display: inline grid;
    place-items: stretch;

    > * {
        grid-area: 1/1;

        border-radius: 0.75em;
    }
}

keyboard-key-underlay {
    transform: translateY(0.25em);

    background: oklch(0.5 0.05 350);
}

kbd {
    position: relative;

    padding: 0.0625em 0.5em;

    background: oklch(0.95 0.08 320);
    border: 1px solid oklch(0.5 0.03 10 / 0.5);
    color: oklch(0.4 0.1 0);
    box-shadow: 0 0 8px -4px oklch(0.5 0.05 350) inset;

    transition:
        transform 0.1s cubic-bezier(0, 1, 0.25, 1),
        filter 0.1s cubic-bezier(0, 1, 0.25, 1);

    &.pressed {
        transform: translateY(0.25em);
        filter: brightness(1.25);
    }
}
</style>