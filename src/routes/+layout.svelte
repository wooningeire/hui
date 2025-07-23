<script lang="ts">
    import type { Snippet } from "svelte";
import "./index.scss";

let {
    children,
}: {
    children: Snippet,
} = $props();
</script>

<main>
    {@render children()}
</main>

<style lang="scss">
main {
    width: 100vw;
    height: 100vh;
    padding: 2rem;
}
</style>