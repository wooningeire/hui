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
        onbeforeinput: (event: InputEvent) => void,
        onkeydown: (event: KeyboardEvent) => void,
        onclick: (event: Event) => void,
        contenteditable: "plaintext-only",
        tabindex: 0,
        role: "textbox",
        disabled: boolean,
        "data-trailing-newline"?: "true",
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
    if (disabled) return;

    focused = true;
};

const handleBlur = () => {
    focused = false;

    if (disabled) return;
    if (localValue === value) return;

    if (!valid) {
        localValue = value;
        return;
    }

    onValueChange(localValue);
};

let el: HTMLElement;

const handleKeydownGeneric = (event: KeyboardEvent) => {
    if (disabled) event.preventDefault();
};

const handleBeforeInput = (event: InputEvent) => {
    if (disabled) {
        event.preventDefault();
        return;
    }

    if (!multiline) return;
    if (event.isComposing) return;
    if (event.inputType !== "insertText") return;
    if (event.data === null) return;

    event.preventDefault();
    insertTextAtSelection(event.data);
};

const selectionInInput = (selection: Selection) => {
    const anchorNode = selection.anchorNode;
    const focusNode = selection.focusNode;

    return (
        anchorNode !== null
        && focusNode !== null
        && el.contains(anchorNode)
        && el.contains(focusNode)
    );
};

const removePlaceholderBreaks = () => {
    if (el.textContent?.length === 0) return;

    for (const child of [...el.childNodes]) {
        if (child.nodeName === "BR") child.remove();
    }
};

const insertTextAtSelection = (text: string) => {
    const selection = el.ownerDocument.getSelection();

    if (selection === null) return;
    if (selection.rangeCount === 0) return;
    if (!selectionInInput(selection)) return;

    const range = selection.getRangeAt(0);
    const textNode = el.ownerDocument.createTextNode(text);

    range.deleteContents();
    range.insertNode(textNode);
    range.setStartAfter(textNode);
    range.setEndAfter(textNode);
    removePlaceholderBreaks();

    selection.removeAllRanges();
    selection.addRange(range);

    el.dispatchEvent(new InputEvent("input", {
        bubbles: true,
        data: text,
        inputType: "insertText",
    }));
};

const handleKeydownMultiline = (event: KeyboardEvent) => {
    handleKeydownGeneric(event);

    if (event.defaultPrevented) return;
    if (event.key !== "Enter") return;

    // Chromium adds a caret placeholder newline for its default plaintext-only Enter behavior.
    event.preventDefault();
    insertTextAtSelection("\n");
};

const handleKeydownSingleLine = (event: KeyboardEvent) => {
    handleKeydownGeneric(event);

    if (event.defaultPrevented) return;

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
        bind:textContent={() => localText, onLocalTextChange}
        {...elProps}
        contenteditable="plaintext-only"
    >
        <br /> <!-- needed for initial centering -->
    </text-entry>
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
            onbeforeinput: handleBeforeInput,
            onkeydown: multiline ? handleKeydownMultiline : handleKeydownSingleLine,
            onclick: (event: Event) => {
                if (!focused) return;
                event.preventDefault();
                event.stopPropagation();
            },
            contenteditable: "plaintext-only",
            tabindex: 0,
            role: "textbox",
            disabled,
            "data-trailing-newline": localValue.endsWith("\n") ? "true" : undefined,
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
        outline-offset: 0.25em;

        color: oklch(62.828% 0.20996 13.579);
    }

    &.disabled {
        pointer-events: none;
    }
}

text-entry {
    display: block;

    white-space: pre-wrap;

    &[data-trailing-newline="true"]::after {
        content: "\200b";
    }
}

text-entry-placeholder {
    opacity: 0.3333333;
    pointer-events: none;
    user-select: none;
}
</style>
