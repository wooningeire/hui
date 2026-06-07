<script lang="ts">
import TextInput from "$lib/TextInput/index.js";

let {
    value = $bindable(),
    placeholderText = "",
    multiline = false,
    disabled = false,
    id = null,
}: {
    value: string,
    placeholderText?: string,
    multiline?: boolean,
    disabled?: boolean,
    id?: string | null,
} = $props();
</script>

<TextInput
    {value}
    onValueChange={newValue => value = newValue}
    {placeholderText}
    {multiline}
    {disabled}
>
    {#snippet container({contents, valid})}
        <text-input-container
            class:invalid={!valid}
            class:disabled
        >
            {@render contents()}
        </text-input-container>
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
            contenteditable="plaintext-only"
            {id}
        >
            <br />
        </div>
    {/snippet}
</TextInput>

<style lang="scss">
text-input-container {
    display: grid;
    place-items: stretch;

    border: 0.0625rem solid oklch(0 0 0 / 0);

    &,
    > * {
        border-radius: 0.5rem;
    }

    &:not(.disabled) {
        box-shadow: 0 0.0625rem 0.125rem oklch(0 0 0 / 0.1);
        border-color: oklch(0 0 0 / 0.25);
    }

    &.disabled {
        pointer-events: none;
    }

    > * {
        grid-area: 1/1;
    }

    &.invalid {
        outline: 1px solid oklch(62.828% 0.20996 13.579);
        outline-offset: 0.25em;

        color: oklch(62.828% 0.20996 13.579);
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
}

.text-input-input {
    white-space: pre-wrap;
}
</style>
