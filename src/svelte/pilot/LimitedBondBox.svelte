<script lang="ts">
    import type { LimitedBondBoxProps } from "@/interfaces/pilot/LimitedBondBoxProps";
    import type { TextLogEventProps } from "@/interfaces/actor/TextLogEventProps";
    import { getLocalized } from "@/scripts/helpers";
    import { resetTextConsole, sendToTextConsole } from "@/scripts/store/text-log";
    import { getActorContext } from "@/scripts/context";
    import { getExtraEffectsEnabled } from "@/scripts/settings";

    const {
        bond,
        power,

        logText,
        logging,
    }: LimitedBondBoxProps & TextLogEventProps = $props();
    const actorUuid = getActorContext()?.uuid;

    const qualityMode = getExtraEffectsEnabled();
    const log = $derived(logText || getLocalized("LA.limited.tooltip"));

    function handlePowerUse(event: Event, index: number)
    {
        event.stopPropagation();

        const path = `system.powers.${power.index}`;
        const val = power.uses.value >= index + 1
            ? Math.max(power.uses.value - 1, power.uses.min)
            : Math.min(power.uses.value + 1, power.uses.max);

        console.log(bond, power.uses.value, power.uses.min, power.uses.max, index + 1);
        bond.update({ [`${path}.uses.value`]: val });
    }
</script>

{#if power.uses}
<div class="la-limited la-flexrow la-text-header 
        -aligncenter -padding1-lr">
    <span class="la-hexarray__span -fontsizemedium">
        {getLocalized("LA.limited.label")}
    </span>
{#each {length: power.uses.max} as _, index}
    <button type="button" 
        class="la-prmy-header la-scdy-primary -fontsize7
            {qualityMode ? '-glow-prmy -glow-scdy-hover' : ''}"
        onpointerenter={ logging ? event => sendToTextConsole(event, log, actorUuid!) : undefined }
        onpointerleave={ logging ? event => resetTextConsole(event, actorUuid!) : undefined }
        onclick={e => handlePowerUse(e, index)}
        aria-label={getLocalized("LA.use.label")}
    >
        <i class="mdi {index < power.uses.value ? 'mdi-hexagon-slice-6' : 'mdi-hexagon-outline'}"></i>
    </button>
{/each}
</div>
{/if}