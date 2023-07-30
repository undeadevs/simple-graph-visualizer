<script lang="ts">
    import { spring, tweened } from 'svelte/motion';
    import { derived } from 'svelte/store';
    import { draw } from 'svelte/transition';

    export let x1 = 0;
    export let y1 = 0;
    export let x2 = 0;
    export let y2 = 0;
    export let r = 0;

    let x1Proc = spring(x1);
    let y1Proc = spring(y1);
    let x2Proc = spring(x2);
    let y2Proc = spring(y2);
    let rProc = tweened(r);

    $: x1Proc.set(x1);
    $: y1Proc.set(y1);
    $: x2Proc.set(x2);
    $: y2Proc.set(y2);
    $: rProc.set(r);

    let dString = derived(
        [x1Proc, y1Proc, x2Proc, y2Proc, rProc],
        ([$x1Val, $y1Val, $x2Val, $y2Val, $rVal]) => {
            if (
                Math.floor($x1Val) === Math.floor($x2Val) &&
                Math.floor($y1Val) === Math.floor($y2Val)
            ) {
                return `M ${$x1Val - r},${$y1Val} a ${$rVal / 2} ${$rVal / 2} 45 1 1 ${
                    $rVal * Math.cos(-Math.PI / 4) - $rVal / 2.5
                },${$rVal * Math.sin(-Math.PI / 4)}`;
            }

            return `M ${$x1Val},${$y1Val} L ${$x2Val},${$y2Val}`;
        }
    );
</script>

<path
    d={$dString}
    fill="none"
    stroke="black"
    marker-end="url(#arrowhead)"
    transition:draw|global={{ duration: 500 }}
/>
