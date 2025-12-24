<script lang="ts">
import TextInput from "$lib/TextInput/index.js";
    import ComponentPage from "../../ComponentPage.svelte";
    import ComponentProps from "../../ComponentProps.svelte";

let value = $state("");
let placeholderText = $state("Placeholder text");
let multiline = $state(false);
let disabled = $state(false);

</script>

<ComponentPage
    title="TextInput"
>

    <ComponentProps>
        <div>
            <label>
                <code>value</code>
                <input
                    type="text"
                    bind:value
                />
            </label>
        </div>

        <div>
            <label>
                <code>placeholderText</code>
                <input
                    type="text"
                    bind:value={placeholderText}
                />
            </label>
        </div>

        <div>
            <label>
                <code>multiline</code>
                <input
                    type="checkbox"
                    bind:checked={multiline}
                />
            </label>
        </div>

        <div>
            <label>
                <code>disabled</code>
                <input
                    type="checkbox"
                    bind:checked={disabled}
                />
            </label>
        </div>
    </ComponentProps>

    <TextInput
        {value}
        onValueChange={newValue => value = newValue}
        {placeholderText}
        {multiline}
        {disabled}
    >
        {#snippet container({contents, valid})}
            <div
                class="text-input-container"
                class:invalid={!valid}
                class:disabled
            >
                {@render contents()}
            </div>
        {/snippet}

        {#snippet placeholder({placeholderText})}
            <div class="text-input-placeholder">{placeholderText}</div>
        {/snippet}

        {#snippet input({localText, onLocalTextChange, el, onElChange, elProps})}
            <div
                bind:this={() => el, onElChange}
                bind:textContent={() => localText, onLocalTextChange}
                class="text-input-input"
                {...elProps}
                contenteditable
            >
                <br />
            </div>
        {/snippet}
    </TextInput>
</ComponentPage>


<style lang="scss">
.text-input-container {
    width: 80ch;

    display: grid;
    place-items: stretch;

    &,
    > * {
        border-radius: 0.5rem;
    }

    &:not(.disabled) {
        box-shadow: 0 0.0625rem 0.125rem oklch(0 0 0 / 0.1);
        border: 0.0625rem solid oklch(0 0 0 / 0.25);
    }

    &.disabled {
        pointer-events: none;
    }

    > * {
        grid-area: 1/1;
    }

    &.invalid {
        border-color: oklch(0.7 0.15 0.98turn);
    }
}

.text-input-placeholder {
    opacity: 0.3333333;
    pointer-events: none;
    user-select: none;
}

.text-input-input,
.text-input-placeholder {
    padding: 0.5rem;
    border-radius: 0.5rem;

    text-align: center;
}
</style>