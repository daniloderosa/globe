<script>
    export let data;
    export let colorScale;

    import { format } from "d3-format";
    import { fade } from "svelte/transition";
    const suffixFormat = (d) => format(".2~s")(d).replace(/G/, "B");

    let minColor = colorScale.range()[0];
    let maxColor = colorScale.range()[1];

    let minValue = colorScale.domain()[0];
    let maxValue = colorScale.domain()[1];

    $: percentOfMax = (data?.population / colorScale.domain()[1]) * 100;
    $: console.log(percentOfMax);
</script>

<div class="legend">
    <span class="label">
        {minValue}
    </span>
    <div
        class="bar"
        style="background:linear-gradient(to right, {minColor}, {maxColor})"
    >
    {#if percentOfMax}
    <span class="line" transition:fade style="left: {percentOfMax}%;" />
    {/if}
</div>
    <span class="label">
        {suffixFormat(maxValue)}
    </span>
</div>

<style>
    .bar {
        height: 15px;
        width: 100%;
        position: relative;
    }

    .legend {
        margin-top: 10px;
        display: flex;
        gap: 6px;
    }

    .label {
        color: white;
        font-size: 0.85rem;
        user-select: none;
    }

    .line {
  position: absolute;
  top: 0;
  height: 15px;
  width: 2px;
  background: white;
  transition: left 800ms cubic-bezier(1, 0, 0, 1);
}
</style>
