<script lang="ts">
    import { spring, tweened } from 'svelte/motion';

    export let x = 0;
    export let y = 0;
    export let r = 0;
    export let name = '_';

    let xProc = spring(0);
    let yProc = spring(0);
    let rProc = tweened(0);

    $: xProc.set(x);
    $: yProc.set(y);
    $: rProc.set(r);

    let strokeColor = 'black';

    function handleMouseEnter(e: Event) {
        strokeColor = 'rgb(6 182 212 / 1)';
    }

    function handleMouseLeave(e: Event) {
        strokeColor = 'black';
    }
</script>

<g
    transform="translate({$xProc} {$yProc})"
    class="focus:outline-none"
    data-name={name}
    on:mouseenter={handleMouseEnter}
    on:mouseleave={handleMouseLeave}
    on:mousedown
    on:mouseup
    on:mousemove
>
    <circle fill="white" stroke={strokeColor} r={Math.abs($rProc)} />
    <text text-anchor="middle" fill={strokeColor} class="pointer-events-none select-none"
        >{name}</text
    >
</g>
