<script lang="ts">
import ComponentPage from "../../ComponentPage.svelte";
import ComponentProp from "../../ComponentProp.svelte";
import ComponentProps from "../../ComponentProps.svelte";
import KeyboardKey from "../../KeyboardKey.svelte";
import TextInputDefaultStyle from "../../TextInputDefaultStyle.svelte";

let value = $state("");
let placeholderText = $state("Placeholder text");
let multiline = $state(false);
let disabled = $state(false);

</script>

<ComponentPage
    title="TextInput"
>
    {#snippet overview()}
        Text entry that only submits changes to the text on blur/enter, not as the user is typing
    {/snippet}

    {#snippet propsSummary()}
        <ComponentProps>
            <ComponentProp
                name="value"
                type="string"
            >
                {#snippet editor({id})}
                    <TextInputDefaultStyle
                        bind:value
                        placeholderText="value"
                        {multiline}
                        {id}
                    />
                {/snippet}

                {#snippet desc()}
                    Last submitted value
                {/snippet}
            </ComponentProp>

            <ComponentProp
                name="placeholderText"
                type="string"
            >
                {#snippet editor({id})}
                    <TextInputDefaultStyle
                        bind:value={placeholderText}
                        placeholderText="placeholderText"
                    />
                {/snippet}

                {#snippet desc()}
                    Text to display when <code>value</code> is empty
                {/snippet}
            </ComponentProp>

            <ComponentProp
                name="multiline"
                type="boolean"
            >
                {#snippet editor()}
                    <input
                        type="checkbox"
                        bind:checked={multiline}
                    />
                {/snippet}

                {#snippet desc()}
                    Whether <KeyboardKey label="Enter" /> should create newlines within the text input (else it will submit the value)
                {/snippet}
            </ComponentProp>

            <ComponentProp
                name="disabled"
                type="boolean"
            >
                {#snippet editor()}
                    <input
                        type="checkbox"
                        bind:checked={disabled}
                    />
                {/snippet}

                {#snippet desc()}
                    Whether to reject further input from the user
                {/snippet}
            </ComponentProp>
        </ComponentProps>
    {/snippet}

    <TextInputDefaultStyle
        bind:value
        {placeholderText}
        {multiline}
        {disabled}
    />
</ComponentPage>


<style lang="scss">
</style>