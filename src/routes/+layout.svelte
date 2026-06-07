<script lang="ts">
    import type { Snippet } from "svelte";
import "./index.scss";
    import ComponentsNav from "./ComponentsNav.svelte";

let {
    children,
}: {
    children: Snippet,
} = $props();
</script>

<main>
    <ComponentsNav />

    <page-content>
        {@render children()}
    </page-content>
</main>

<style lang="scss">
main {
    display: flex;
    gap: 2em;
    
    width: 100vw;
    height: 100vh;

    overflow: hidden;

    :global(> *) {
        overflow-y: auto;
    }
}

page-content {
    flex-grow: 1;

    padding: 2em;
}
</style>