<script lang="ts">
import EntrySlider from "$lib/EntrySlider/index.js";
import ComponentPage from "../../ComponentPage.svelte";
import ComponentProps from "../../ComponentProps.svelte";

let value = $state(0.5);
let min = $state(0);
let max = $state(1);
let softMin = $state(0);
let softMax = $state(1);
let step = $state(0.01);
let hasBounds = $state(true);
let disabled = $state(false);

const clampProgress = (value: number) => {
    return Math.max(0, Math.min(1, value));
};
</script>

<ComponentPage
    title="EntrySlider"
>
    {#snippet propsSummary()}
        <ComponentProps>
            <code>value</code>
            <input
                type="number"
                step={0.01}
                bind:value
            />

            <div>
                <label>
                    <code>min</code>
                    <input
                        type="number"
                        step={0.01}
                        bind:value={min}
                    />
                </label>
            </div>

            <div>
                <label>
                    <code>max</code>
                    <input
                        type="number"
                        step={0.01}
                        bind:value={max}
                    />
                </label>
            </div>

            <div>
                <label>
                    <code>softMin</code>
                    <input
                        type="number"
                        step={0.01}
                        bind:value={softMin}
                    />
                </label>
            </div>

            <div>
                <label>
                    <code>softMax</code>
                    <input
                        type="number"
                        step={0.01}
                        bind:value={softMax}
                    />
                </label>
            </div>

            <div>
                <label>
                    <code>step</code>
                    <input
                        type="number"
                        step={0.001}
                        bind:value={step}
                    />
                </label>
            </div>

            <div>
                <label>
                    <code>hasBounds</code>
                    <input
                        type="checkbox"
                        bind:checked={hasBounds}
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
    {/snippet}

    <EntrySlider
        {value}
        onValueChange={newValue => value = newValue}
        {min}
        {max}
        {softMin}
        {softMax}
        {step}
        {hasBounds}
        {disabled}
    >
        {#snippet entry({
            text,
            el,
            onElChange,
            elProps,
            valid,
            editing,
            dragging,
            overflow,
            underflow,
            progress,
        })}
            <input
                bind:this={() => el, onElChange}
                class="entry-slider"
                class:dragging
                class:editing
                class:invalid={!valid}
                class:overflow
                class:underflow
                style:--slider-progress={clampProgress(progress)}
                {...elProps}
                value={text}
            />
        {/snippet}
    </EntrySlider>
</ComponentPage>

<style lang="scss">
.entry-slider {
    width: 16ch;
    padding: 0.375rem 0.5rem;
    border: 0.0625rem solid oklch(0 0 0 / 0.25);
    border-radius: 0.25rem;
    box-shadow: 0 0.0625rem 0.125rem oklch(0 0 0 / 0.1);
    cursor: ew-resize;
    text-align: right;
    background:
        linear-gradient(
            90deg,
            oklch(82% 0.11 170 / 0.45) calc(var(--slider-progress) * 100%),
            transparent 0
        ),
        white;

    &.editing {
        cursor: text;
    }

    &.dragging {
        border-color: oklch(54% 0.14 240);
    }

    &.invalid {
        border-color: oklch(62.828% 0.20996 13.579);
    }

    &.overflow,
    &.underflow {
        color: oklch(45% 0.15 35);
    }
}
</style>
