<script lang="ts">
import type { Snippet } from "svelte";

let {
    children,
}: {
    children: Snippet,
} = $props();
</script>

<component-props>
    {@render children()}
</component-props>

<style lang="scss">
component-props {
    align-self: center;

    display: grid;
    gap: 2em 1em;
    grid-template-columns: 1fr repeat(3, auto);
    align-items: center;

    width: 100ch;
    padding: 1rem;

    border: 1px solid oklch(0.8 0.05 130);
    border-radius: 1.5em;

    box-shadow: 0 0 16px -8px oklch(0.4 0.08 180);
}
</style>