<script>
    // export let id = "test";
    export let content = "content should be here";
    export let name;
    export let network;
    export let shop;
    export let layer;
    export let properties;
</script>

<p style="white-space: break-spaces; max-height: 150px; overflow: auto; padding: 5px;">
{#if network}
{network}
<br>
{/if}
{#if name}
{name}
{:else}
Transit Station
{/if}
</p>

