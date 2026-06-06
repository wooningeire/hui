<script lang="ts">
import type { Snippet } from "svelte";

let {
    children,
    overview,
    propsSummary,
    title,
}: {
    children: Snippet,
    overview: Snippet,
    propsSummary: Snippet,
    title: string,
} = $props();
</script>

<component-page>
    <h1>{title}</h1>

    {@render overview()}

    <h2>Props</h2>
    {@render propsSummary()}


    <h2>Preview</h2>
    <div class="component-container">
        {@render children()}
    </div>
</component-page>

<style lang="scss">
component-page {
    display: flex;
    flex-direction: column;

    gap: 1rem;
}
</style>