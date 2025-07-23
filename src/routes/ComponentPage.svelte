<script lang="ts">
import type { Snippet } from "svelte";

let {
    children,
    title,
}: {
    children: Snippet,
    title: string,
} = $props();
</script>

<h1>{title}</h1>

<div class="component-container">
    {@render children()}
</div>