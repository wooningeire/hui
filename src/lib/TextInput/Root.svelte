<script lang="ts">
import {type Snippet} from "svelte";

export type ContainerProps = {
    contents: Snippet,
    localText: string,
    focused: boolean,
    disabled: boolean,
    valid: boolean,
};

export type ContentsProps = {
    placeholder: Snippet,
    input: Snippet,
    localText: string,
    focused: boolean,
    disabled: boolean,
    valid: boolean,
};

export type PlaceholderProps = {
    placeholderText: string,
    localText: string,
    focused: boolean,
    disabled: boolean,
    valid: boolean,
};

export type InputProps = {
    localText: string,
    onLocalTextChange: (value: string) => void,
    focused: boolean,
    disabled: boolean,
    valid: boolean,
    el: HTMLElement,
    onElChange: (el: HTMLElement) => void,
    elProps: {
        onfocus: () => void,
        onblur: () => void,
        onkeydown: (event: KeyboardEvent) => void,
        onclick: (event: Event) => void,
        contenteditable: true,
        tabindex: 0,
        role: "textbox",
        disabled: boolean,
    },
};

let {
    value,
    onValueChange,
    container = null,
    contents = null,
    placeholder = null,
    input = null,
    placeholderText = null,
    validate = () => true,
    disabled = false,
    multiline = false,
}: {
    value: string,
    onValueChange: (value: string) => void,
    container?: Snippet<[ContainerProps]> | null,
    contents?: Snippet<[ContentsProps]> | null,
    input?: Snippet<[InputProps]> | null,
    placeholder?: Snippet<[PlaceholderProps]> | null,
    placeholderText?: string | null,
    validate?: (value: string) => boolean,
    disabled?: boolean,
    multiline?: boolean,
} = $props();


let localValue = $state(value);
const valid = $derived(validate(localValue));

$effect(() => {
    localValue = value;
});


let focused = $state(false);
const handleFocus = () => {
    focused = true;
};

const handleBlur = () => {
    focused = false;

    if (localValue === value) return;

    if (!valid) {
        localValue = value;
        return;
    }

    onValueChange(localValue);
};

let el: HTMLElement;
const handleKeydownSingleLine = (event: KeyboardEvent) => {
    if (event.key === "Enter") {
        event.preventDefault();
        el.blur();
    }
};

const containerFn = $derived(container ?? containerDefault);
const contentsFn = $derived(contents ?? contentsDefault);
const placeholderFn = $derived(placeholder ?? placeholderDefault);
const inputFn = $derived(input ?? inputDefault);
</script>

{#snippet containerDefault({
    contents,
    disabled,
    valid,
}: ContainerProps)}
    <text-entry-container
        class:invalid={!valid}
        class:disabled
    >
        {@render contents()}
    </text-entry-container>
{/snippet}

{#snippet contentsDefault({
    placeholder,
    input,
    localText,
}: ContentsProps)}
    {#if localText.length === 0}
        {@render placeholder()}
    {/if}

    {@render input()}
{/snippet}

{#snippet placeholderDefault({
    placeholderText,
}: PlaceholderProps)}
    <text-entry-placeholder>{placeholderText}</text-entry-placeholder>
{/snippet}

{#snippet inputDefault({
    localText,
    onLocalTextChange,
    el,
    onElChange,
    elProps,
}: InputProps)}
    <text-entry
        bind:this={() => el, onElChange}
        contenteditable
        role="textbox"
        tabindex="0"
        {onfocus}
        {onblur}
        {onkeydown}
        {onclick}
        bind:textContent={() => localText, onLocalTextChange}
        {disabled}
    ></text-entry>
{/snippet}

{#snippet placeholderLoaded()}
    {@render placeholderFn({
        placeholderText: placeholderText ?? "",
        localText: localValue,
        focused,
        disabled,
        valid,
    })}
{/snippet}

{#snippet inputLoaded()}
    {@render inputFn({
        localText: localValue,
        onLocalTextChange: (value: string) => {
            localValue = value;
        },
        focused,
        disabled,
        valid,
        el,
        onElChange: (value: HTMLElement) => {
            el = value;
        },
        elProps: {
            onfocus: handleFocus,
            onblur: handleBlur,
            onkeydown: multiline ? () => {} : handleKeydownSingleLine,
            onclick: (event: Event) => {
                if (!focused) return;
                event.preventDefault();
                event.stopPropagation();
            },
            contenteditable: true,
            tabindex: 0,
            role: "textbox",
            disabled,
        },
    })}
{/snippet}

{#snippet contentsLoaded()}
    {@render contentsFn({
        placeholder: placeholderLoaded,
        input: inputLoaded,
        localText: localValue,
        focused,
        disabled,
        valid,
    })}
{/snippet}


{@render containerFn({
    contents: contentsLoaded,
    localText: localValue,
    focused,
    disabled,
    valid,
})}


<style lang="scss">
text-entry-container {
    display: grid;
    place-items: stretch;

    > * {
        grid-area: 1/1;
    }

    &.invalid {
        outline: 1px solid oklch(62.828% 0.20996 13.579);
        outline-offset: 0.25rem;
    }
}

text-entry {
    display: block;
}

text-entry-placeholder {
    opacity: 0.3333333;
    pointer-events: none;
    user-select: none;
}
</style>