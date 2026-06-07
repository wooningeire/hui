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
    padding: 2em 4em;
}
</style>