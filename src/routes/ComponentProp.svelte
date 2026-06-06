<script lang="ts">
import type { Snippet } from "svelte";

const {
    name,
    type = null,
    desc,
    editor,
}: {
    name: string,
    type?: string | null,
    desc: Snippet,
    editor: Snippet<[{
        id: string,
    }]>,
} = $props();

const id = $props.id();
</script>

<component-prop>
    {@render editor({id})}

    <label for={id}>
        <code>{name}</code>
    </label>
        
    <component-prop-type-information>
        {#if type !== null}
            : <code>{type}</code>
        {/if}
    </component-prop-type-information>

    <component-prop-desc>
        {@render desc()}
    </component-prop-desc>
</component-prop>

<style lang="scss">
component-prop {
    display: contents;
}

component-prop-type-information {
    font-size: 0.75em;
}

component-prop-desc {
    max-width: 40ch;
}
</style>