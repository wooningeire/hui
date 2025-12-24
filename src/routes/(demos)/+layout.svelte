<script lang="ts">
    import type { Snippet } from "svelte";

let {
    children,
}: {
    children: Snippet,
} = $props();
</script>

<div>
    <a href="/">back</a>
</div>

{@render children()}