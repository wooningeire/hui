<script lang="ts">
import EntrySlider from "$lib/EntrySlider/index.js";
import ComponentPage from "../../ComponentPage.svelte";
import ComponentProp from "../../ComponentProp.svelte";
import ComponentProps from "../../ComponentProps.svelte";

let value = $state(1);
let min = $state(0.01);
let max = $state(100);
let softMin = $state(0.01);
let softMax = $state(100);
let step = $state(0.01);
let exponential = $state(true);
let hasBounds = $state(true);
let disabled = $state(false);

const clampProgress = (value: number) => {
    return Math.max(0, Math.min(1, value));
};
</script>

<ComponentPage
    title="EntrySlider"
>
    {#snippet overview()}
        Numeric entry that drags like a slider and clicks into text entry
    {/snippet}

    {#snippet propsSummary()}
        <ComponentProps>
            <ComponentProp
                name="value"
                type="number"
            >
                {#snippet editor({id})}
                    <input
                        {id}
                        type="number"
                        step={0.01}
                        bind:value
                    />
                {/snippet}

                {#snippet desc()}
                    Current numeric value
                {/snippet}
            </ComponentProp>

            <ComponentProp
                name="min"
                type="number"
            >
                {#snippet editor({id})}
                    <input
                        {id}
                        type="number"
                        step={0.01}
                        bind:value={min}
                    />
                {/snippet}

                {#snippet desc()}
                    Smallest allowed value accepted from either text or drag input
                {/snippet}
            </ComponentProp>

            <ComponentProp
                name="max"
                type="number"
            >
                {#snippet editor({id})}
                    <input
                        {id}
                        type="number"
                        step={0.01}
                        bind:value={max}
                    />
                {/snippet}

                {#snippet desc()}
                    Largest allowed value accepted from either text or slider input
                {/snippet}
            </ComponentProp>

            <ComponentProp
                name="softMin"
                type="number"
            >
                {#snippet editor({id})}
                    <input
                        {id}
                        type="number"
                        step={0.01}
                        bind:value={softMin}
                    />
                {/snippet}

                {#snippet desc()}
                    Lower value used for slider progress and snapping
                {/snippet}
            </ComponentProp>

            <ComponentProp
                name="softMax"
                type="number"
            >
                {#snippet editor({id})}
                    <input
                        {id}
                        type="number"
                        step={0.01}
                        bind:value={softMax}
                    />
                {/snippet}

                {#snippet desc()}
                    Upper value used for slider progress and snapping
                {/snippet}
            </ComponentProp>

            <ComponentProp
                name="step"
                type="number"
            >
                {#snippet editor({id})}
                    <input
                        {id}
                        type="number"
                        step={0.001}
                        bind:value={step}
                    />
                {/snippet}

                {#snippet desc()}
                    Rounding increment used while dragging
                {/snippet}
            </ComponentProp>

            <ComponentProp
                name="exponential"
                type="boolean"
            >
                {#snippet editor({id})}
                    <input
                        {id}
                        type="checkbox"
                        bind:checked={exponential}
                    />
                {/snippet}

                {#snippet desc()}
                    Whether <code>value</code> rises exponentially with slider progress
                {/snippet}
            </ComponentProp>

            <ComponentProp
                name="hasBounds"
                type="boolean"
            >
                {#snippet editor({id})}
                    <input
                        {id}
                        type="checkbox"
                        bind:checked={hasBounds}
                    />
                {/snippet}

                {#snippet desc()}
                    Whether soft bounds define slider progress and drag speed
                {/snippet}
            </ComponentProp>

            <ComponentProp
                name="disabled"
                type="boolean"
            >
                {#snippet editor({id})}
                    <input
                        {id}
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

    <EntrySlider
        {value}
        onValueChange={newValue => value = newValue}
        {min}
        {max}
        {softMin}
        {softMax}
        {step}
        {exponential}
        {hasBounds}
        {disabled}
    >
        {#snippet entry({
            text,
            el,
            onElChange,
            elProps,
            outsideHardBounds,
            outsideSoftBounds,
            belowSoftMax,
            belowSoftMin,
            editing,
            dragging,
            progress,
        })}
            <input
                bind:this={() => el, onElChange}
                class="entry-slider"
                class:dragging
                class:editing
                class:outside-hard-bounds={outsideHardBounds}
                class:outside-soft-bounds={outsideSoftBounds}
                class:above-soft-max={belowSoftMax}
                class:below-soft-min={belowSoftMin}
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

    
    text-align: right;

    background:
        linear-gradient(
            90deg,
            oklch(0.82 0.11 170 / 0.45) calc(var(--slider-progress) * 100%),
            oklch(0 0 0 / 0) 0
        ),
        white;

    cursor: ew-resize;

    &.editing {
        cursor: text;
    }

    &.outside-hard-bounds {
        outline: 1px solid oklch(62.828% 0.20996 13.579);
        outline-offset: 0.25em;

        color: oklch(62.828% 0.20996 13.579);
    }

    &.below-soft-min {
        background:
            linear-gradient(
                90deg,
                oklch(0.9 0.15 150 / 0.7),
                oklch(0 0 0 / 0) 25%
            ),
    }

    &.above-soft-max {
        background:
            linear-gradient(
                90deg,
                oklch(0.82 0.11 170 / 0.45) 75%,
                oklch(0.95 0.15 150 / 0.7)
            ),
    }

}
</style>
