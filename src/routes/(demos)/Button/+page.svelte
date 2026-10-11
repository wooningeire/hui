<script lang="ts">
    import { Button } from "$lib/index.js";
import ComponentPage from "../../ComponentPage.svelte";
    import ComponentProps from "../../ComponentProps.svelte";
</script>

<ComponentPage
    title="Button"
>
    {#snippet overview()}
        Button that separates click area and display area
    {/snippet}

    {#snippet propsSummary()}
        <ComponentProps>
            TBD
        </ComponentProps>
    {/snippet}
    
    <Button
        --bg-col="oklch(1 0 0)"
    >
        Hello
    </Button>
</ComponentPage>