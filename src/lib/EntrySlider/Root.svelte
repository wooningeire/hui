<script lang="ts">
import {tick, type Snippet} from "svelte";

export type EntrySliderMode = "idle" | "pending-drag" | "dragging" | "editing";

export type EntryProps = {
    text: string,
    value: number,
    progress: number,
    valid: boolean,
    focused: boolean,
    disabled: boolean,
    editing: boolean,
    dragging: boolean,
    pendingDrag: boolean,
    overflow: boolean,
    underflow: boolean,
    hasBounds: boolean,
    el: HTMLElement | null,
    onElChange: (el: HTMLElement | null) => void,
    onTextChange: (value: string) => void,
    startTextEntry: () => void,
    stopTextEntry: () => void,
    elProps: {
        value: string,
        type: "text",
        inputmode: "decimal",
        role: "spinbutton",
        disabled: boolean,
        "aria-disabled": boolean,
        "aria-invalid": boolean,
        "aria-valuemax": number | undefined,
        "aria-valuemin": number | undefined,
        "aria-valuenow": number,
        onblur: () => void,
        onchange: () => void,
        onclick: (event: MouseEvent) => void,
        onfocus: () => void,
        oninput: (event: Event) => void,
        onkeydown: (event: KeyboardEvent) => void,
        onpointerdown: (event: PointerEvent) => void,
    },
};

type Props = {
    value: number,
    onValueChange: (value: number) => void,
    entry?: Snippet<[EntryProps]> | null,
    validate?: (value: number) => boolean,
    convertIn?: (value: number) => number,
    convertOut?: (value: number) => number,
    format?: (value: number) => string,
    parse?: (value: string) => number | null,
    hasBounds?: boolean,
    min?: number,
    max?: number,
    softMin?: number,
    softMax?: number,
    step?: number,
    unboundedChangePerPixel?: number,
    dragTolerance?: number,
    stickToBoundTolerance?: number,
    usePointerLock?: boolean,
    disabled?: boolean,
};

type Point = {
    x: number,
    y: number,
};

type DragState = {
    pointerId: number,
    start: Point,
    totalX: number,
    startValue: number,
};

const identity = (value: number) => value;
const acceptAlways = () => true;

const defaultFormat = (value: number) => {
    if (!Number.isFinite(value)) return value.toString();

    return Number(Number(value).toFixed(4)).toString();
};

const defaultParse = (value: string) => {
    const trimmed = value.trim();
    if (trimmed.length === 0) return null;

    const parsed = Number(trimmed);
    if (!Number.isFinite(parsed)) return null;

    return parsed;
};

const clamp = (value: number, min: number, max: number) => {
    return Math.max(min, Math.min(max, value));
};

const getProgress = (value: number, min: number, max: number) => {
    const span = max - min;
    if (!Number.isFinite(span) || span === 0) return 0;

    return (value - min) / span;
};

const getModifierFactor = (event: PointerEvent) => {
    if (event.shiftKey) return 1 / 8;
    if (event.ctrlKey || event.metaKey) return 8;

    return 1;
};

const roundToStep = (value: number, step: number) => {
    if (!Number.isFinite(step) || step <= 0) return value;

    const inverseStep = 1 / step;
    if (Number.isInteger(inverseStep)) {
        return Math.round(value * inverseStep) / inverseStep;
    }

    return Math.round(value / step) * step;
};

let {
    value,
    onValueChange,
    entry = null,
    validate = acceptAlways,
    convertIn = identity,
    convertOut = identity,
    format = defaultFormat,
    parse = defaultParse,
    hasBounds = true,
    min = -Infinity,
    max = Infinity,
    softMin = 0,
    softMax = 1,
    step = 1e-3,
    unboundedChangePerPixel = 0.03125,
    dragTolerance = 3,
    stickToBoundTolerance = 12,
    usePointerLock = true,
    disabled = false,
}: Props = $props();

const getDisplayText = () => format(convertOut(value));

let localText = $state("");
let mode = $state("idle" as EntrySliderMode);
let valid = $state(true);
let focused = $state(false);
let el = $state(null as HTMLElement | null);
let dragState = $state.raw(null as DragState | null);
let suppressNextClick = $state(false);

const editing = $derived(mode === "editing");
const pendingDrag = $derived(mode === "pending-drag");
const dragging = $derived(mode === "dragging");
const trackingPointer = $derived(pendingDrag || dragging);
const internalSoftMin = $derived(convertIn(softMin));
const internalSoftMax = $derived(convertIn(softMax));
const progress = $derived(getProgress(value, internalSoftMin, internalSoftMax));
const overflow = $derived(hasBounds && progress > 1);
const underflow = $derived(hasBounds && progress < 0);
const displayText = $derived(editing ? localText : getDisplayText());

const getAmountPerPixel = () => {
    if (!hasBounds) return unboundedChangePerPixel;

    const span = internalSoftMax - internalSoftMin;
    const width = el?.getBoundingClientRect().width ?? 0;

    if (!Number.isFinite(span) || span === 0 || width <= 0) {
        return unboundedChangePerPixel;
    }

    return span / width;
};

const isHardBounded = (value: number) => {
    return value >= min && value <= max;
};

const isValid = (value: number) => {
    return Number.isFinite(value) && isHardBounded(value) && validate(value);
};

$effect(() => {
    if (editing) return;

    localText = getDisplayText();

    if (mode === "idle") {
        valid = isValid(value);
    }
});

const parseTextValue = (text: string) => {
    const parsed = parse(text);
    if (parsed === null) return null;

    const converted = convertIn(parsed);
    if (!Number.isFinite(converted)) return null;

    return converted;
};

const emitIfValid = (nextValue: number) => {
    valid = isValid(nextValue);
    if (!valid) return;

    onValueChange(nextValue);
};

const handleTextChange = (text: string) => {
    if (disabled) return;

    mode = "editing";
    localText = text;

    const nextValue = parseTextValue(text);
    valid = nextValue !== null && isValid(nextValue);

    if (nextValue !== null && valid) {
        onValueChange(nextValue);
    }
};

const stopTextEntry = () => {
    mode = "idle";
    localText = getDisplayText();
    valid = true;
};

const selectElementText = () => {
    if (el === null) return;

    el.focus();

    if (el instanceof HTMLInputElement || el instanceof HTMLTextAreaElement) {
        el.select();
        return;
    }

    const selection = window.getSelection();
    if (selection === null) return;

    const range = document.createRange();
    range.selectNodeContents(el);
    selection.removeAllRanges();
    selection.addRange(range);
};

const startTextEntry = () => {
    if (disabled) return;

    mode = "editing";
    localText = getDisplayText();
    valid = true;

    void tick().then(selectElementText);
};

const clearSelection = () => {
    window.getSelection()?.removeAllRanges();
};

const requestPointerLock = () => {
    if (!usePointerLock) return;
    if (el === null) return;

    const request = el.requestPointerLock?.();
    if (request instanceof Promise) {
        void request.catch(() => {});
    }
};

const exitPointerLock = () => {
    if (document.pointerLockElement === null) return;

    void document.exitPointerLock();
};

const snapToSoftBounds = (nextValue: number, amountPerPixel: number, modifierFactor: number) => {
    if (!hasBounds) {
        return {
            value: nextValue,
            snapped: false,
        };
    }

    const tolerance = Math.abs(amountPerPixel * modifierFactor * stickToBoundTolerance);

    if (Math.abs(nextValue - internalSoftMin) <= tolerance) {
        return {
            value: clamp(internalSoftMin, min, max),
            snapped: true,
        };
    }

    if (Math.abs(nextValue - internalSoftMax) <= tolerance) {
        return {
            value: clamp(internalSoftMax, min, max),
            snapped: true,
        };
    }

    return {
        value: nextValue,
        snapped: false,
    };
};

const emitDragValue = (event: PointerEvent, totalX: number, startValue: number) => {
    const amountPerPixel = getAmountPerPixel();
    const modifierFactor = getModifierFactor(event);
    const rawValue = startValue + totalX * amountPerPixel * modifierFactor;
    const boundedValue = clamp(rawValue, min, max);
    const snappedValue = snapToSoftBounds(boundedValue, amountPerPixel, modifierFactor);
    const steppedValue = snappedValue.snapped
        ? snappedValue.value
        : clamp(roundToStep(snappedValue.value, step), min, max);

    emitIfValid(steppedValue);
};

const shouldIgnorePointer = (event: PointerEvent) => {
    return disabled || event.button !== 0 || editing;
};

const handlePointerDown = (event: PointerEvent) => {
    if (shouldIgnorePointer(event)) return;

    event.preventDefault();
    event.stopPropagation();

    const target = event.currentTarget;
    if (target instanceof HTMLElement) {
        el = target;
    }

    mode = "pending-drag";
    valid = true;
    dragState = {
        pointerId: event.pointerId,
        start: {
            x: event.pageX,
            y: event.pageY,
        },
        totalX: 0,
        startValue: value,
    };
};

const handlePointerMove = (event: PointerEvent) => {
    if (dragState === null) return;
    if (event.pointerId !== dragState.pointerId) return;

    const displacement = {
        x: event.pageX - dragState.start.x,
        y: event.pageY - dragState.start.y,
    };

    if (pendingDrag) {
        const distance = Math.hypot(displacement.x, displacement.y);
        if (distance < dragTolerance) return;

        mode = "dragging";
        requestPointerLock();
    }

    const totalX = document.pointerLockElement === null
        ? displacement.x
        : dragState.totalX + event.movementX;

    dragState = {
        ...dragState,
        totalX,
    };

    emitDragValue(event, totalX, dragState.startValue);
};

const endPointerTracking = () => {
    dragState = null;
    mode = "idle";
    exitPointerLock();
};

const handlePointerUp = (event: PointerEvent) => {
    if (dragState === null) return;
    if (event.pointerId !== dragState.pointerId) return;

    const wasDragging = dragging;
    endPointerTracking();

    event.preventDefault();
    event.stopPropagation();

    suppressNextClick = true;

    if (wasDragging) {
        setTimeout(() => {
            suppressNextClick = false;
            clearSelection();
        }, 0);
        return;
    }

    startTextEntry();
    setTimeout(() => {
        suppressNextClick = false;
    }, 0);
};

const handlePointerCancel = (event: PointerEvent) => {
    if (dragState === null) return;
    if (event.pointerId !== dragState.pointerId) return;

    endPointerTracking();
};

const handleClick = (event: MouseEvent) => {
    if (disabled) return;

    if (suppressNextClick) {
        event.preventDefault();
        event.stopPropagation();
        return;
    }

    if (editing) return;

    event.preventDefault();
    event.stopPropagation();
    startTextEntry();
};

const handleInput = (event: Event) => {
    const target = event.currentTarget;
    if (!(target instanceof HTMLInputElement || target instanceof HTMLTextAreaElement)) return;

    handleTextChange(target.value);
};

const handleFocus = () => {
    if (disabled) return;

    focused = true;
};

const handleBlur = () => {
    focused = false;

    if (!editing) return;
    stopTextEntry();
};

const handleKeydown = (event: KeyboardEvent) => {
    if (disabled) {
        event.preventDefault();
        return;
    }

    if (event.key === "Enter") {
        event.preventDefault();
        stopTextEntry();
        el?.blur();
        return;
    }

    if (event.key === "Escape") {
        event.preventDefault();
        stopTextEntry();
        el?.blur();
    }
};

const entryFn = $derived(entry ?? entryDefault);
</script>

<svelte:window
    onpointermove={trackingPointer ? handlePointerMove : null}
    onpointerup={trackingPointer ? handlePointerUp : null}
    onpointercancel={trackingPointer ? handlePointerCancel : null}
/>

{#snippet entryDefault({
    text,
    el,
    onElChange,
    elProps,
    valid,
    disabled,
    editing,
    dragging,
    overflow,
    underflow,
    progress,
}: EntryProps)}
    <entry-slider
        class:disabled
        class:dragging
        class:editing
        class:invalid={!valid}
        class:overflow
        class:underflow
        style:--entry-slider-progress={progress}
    >
        <input
            bind:this={() => el, onElChange}
            {...elProps}
            value={text}
        />
    </entry-slider>
{/snippet}

{@render entryFn({
    text: displayText,
    value,
    progress,
    valid,
    focused,
    disabled,
    editing,
    dragging,
    pendingDrag,
    overflow,
    underflow,
    hasBounds,
    el,
    onElChange: (value: HTMLElement | null) => {
        el = value;
    },
    onTextChange: handleTextChange,
    startTextEntry,
    stopTextEntry,
    elProps: {
        value: displayText,
        type: "text",
        inputmode: "decimal",
        role: "spinbutton",
        disabled,
        "aria-disabled": disabled,
        "aria-invalid": !valid,
        "aria-valuemax": hasBounds ? max : undefined,
        "aria-valuemin": hasBounds ? min : undefined,
        "aria-valuenow": value,
        onblur: handleBlur,
        onchange: stopTextEntry,
        onclick: handleClick,
        onfocus: handleFocus,
        oninput: handleInput,
        onkeydown: handleKeydown,
        onpointerdown: handlePointerDown,
    },
})}

<style lang="scss">
entry-slider {
    display: inline-grid;
    min-width: 6ch;

    > input {
        min-width: 0;
        width: 100%;
        cursor: ew-resize;
        text-align: right;
    }

    &.editing > input {
        cursor: text;
    }

    &.invalid > input {
        outline: 1px solid oklch(62.828% 0.20996 13.579);
        outline-offset: 0.25rem;
    }

    &.disabled {
        pointer-events: none;
    }
}
</style>
