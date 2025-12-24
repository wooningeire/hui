<script lang="ts">
    import type { Snippet } from "svelte";

    let {
        children,
    }: {
        children: Snippet,
    } = $props();
</script>

<div>
    {@render children()}
</div>

<style lang="scss">
div {
    padding: 1rem;
}
</style>