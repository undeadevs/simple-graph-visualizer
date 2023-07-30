<script lang="ts">
    import GEdge from '$lib/GEdge.svelte';
    import GNode from '$lib/GNode.svelte';

    type GNodePos = [number, number];
    type GNodeName = string & { __brand: 'GNodeName' };
    type GNodeList = Record<GNodeName, GNodePos>;

    let innerWidth = 0;
    let innerHeight = 0;
    let viewerWidth = 0;
    let viewerHeight = 0;

    let edges: [GNodeName, GNodeName][] = [];
    let nodes: GNodeList = {};
    $: nodeRadius = 150 / Object.keys(nodes).length;
    const distFromOrigin = 100;

    function getRecordKeys<T extends Record<string, unknown>, K extends keyof T>(obj: T): K[] {
        return Object.keys(obj) as K[];
    }

    function updateNodes() {
        console.time("nodes")
        let res: GNodeList = {};
        for (let i = 0; i < edges.length; i++) {
            const edge = edges[i];
            if (!(edge[0] in res)) {
                res[edge[0]] = [0, 0];
            }
            if (!(edge[1] in res)) {
                res[edge[1]] = [0, 0];
            }
        }
        const nodeNames = getRecordKeys(res);
        const nodesCount = nodeNames.length;
        const angleDiv = 360 / nodesCount;
        for (let i = 0; i < nodesCount; i++) {
            const angle = i * angleDiv;
            const angleRad = (angle * Math.PI) / 180;
            const x = distFromOrigin * Math.cos(angleRad);
            const y = distFromOrigin * Math.sin(angleRad);
            res[nodeNames[i]] = [x, y];
        }
        console.timeEnd("nodes");
        nodes = res;
    }

    function getLineBetween(node1: GNodePos, node2: GNodePos) {
        const xBetween = node2[0] - node1[0];
        const yBetween = node2[1] - node1[1];
        const magBetween = Math.sqrt(xBetween ** 2 + yBetween ** 2);
        const dx = (xBetween * nodeRadius) / (magBetween || 1);
        const dy = (yBetween * nodeRadius) / (magBetween || 1);
        const x1 = node1[0] + dx;
        const y1 = node1[1] + dy;
        const x2 = node2[0] - dx;
        const y2 = node2[1] - dy;

        return {
            x1,
            y1,
            x2,
            y2
        };
    }

    function handleInput(e: Event, i: number, side: number) {
        edges[i][side] = (e.target as HTMLInputElement).value as GNodeName;
        updateNodes();
    }

    function handleAdd(e: MouseEvent) {
        edges[edges.length] = ['_', '_'] as [GNodeName, GNodeName];
        updateNodes();
    }

    function handleDelete(e: MouseEvent, i: number) {
        edges.splice(i, 1);
        edges = edges;
        updateNodes();
    }

    let isHolding: Record<GNodeName, boolean> = {};

    function handleNodePointerDown(e: Event, nodeName: string) {
        isHolding[nodeName as GNodeName] = true;
    }

    function handleNodePointerUp(e: Event, nodeName: string) {
        isHolding[nodeName as GNodeName] = false;
    }

    function handleNodePointerMove(e: MouseEvent) {
        getRecordKeys(isHolding).forEach((nodeName) => {
            if (isHolding[nodeName]) {
                const updatedX = e.offsetX - viewerWidth / 2;
                const updatedY = e.offsetY - viewerHeight / 2;
                nodes[nodeName] = [updatedX, updatedY];
            }
        });
    }

    function bubbleDownPointerUp(e: MouseEvent) {
        getRecordKeys(isHolding).forEach((nodeName) => {
            if (isHolding[nodeName]) {
                const chosenNodeEl = document.querySelector(`circle[data-name="${nodeName}"]`);
                if (chosenNodeEl === e.target) return;
                isHolding[nodeName] = false;
            }
        });
    }
</script>

<svelte:window bind:innerWidth bind:innerHeight />

<main class="flex flex-col md:flex-row overflow-hidden w-screen h-screen">
    <aside
        class="w-screen min-w-screen h-[15rem] min-h-[15rem] md:w-[24rem] md:min-w-[24rem] md:h-screen md:min-h-screen bg-gray-600 overflow-auto p-6 grid gap-2 auto-rows-max"
    >
        <h3 class="text-xl font-extrabold text-white">Simple Graph Visualizer</h3>
        <button
            class="bg-cyan-500 text-white hover:shadow-[0_0_1rem] hover:shadow-cyan-500 transition-shadow p-2 rounded"
            on:click={handleAdd}
            ><i class="fa-solid fa-add" /> <span class="font-bold">Add</span></button
        >
        <ul class="mt-4 grid gap-3 auto-rows-max">
            {#each edges as edge, i (i)}
                <li class="flex gap-2">
                    <span class="flex-1 flex gap-2 items-center">
                        <input
                            type="text"
                            class="p-2 rounded flex-1 w-full md:max-w-[8rem] bg-gray-700 text-gray-100"
                            value={edge[0]}
                            on:input={(e) => handleInput(e, i, 0)}
                        />
                        <span class="text-gray-300 font-bold"
                            ><i class="fa-solid fa-arrow-right" /></span
                        >
                        <input
                            type="text"
                            class="p-2 rounded flex-1 w-full md:max-w-[8rem] bg-gray-700 text-gray-100"
                            value={edge[1]}
                            on:input={(e) => handleInput(e, i, 1)}
                        />
                    </span>
                    <button
                        class="bg-red-500 text-white hover:shadow-[0_0_1rem] hover:shadow-red-500 transition-shadow fond-bold p-2 rounded"
                        on:click={(e) => handleDelete(e, i)}><i class="fa-solid fa-trash" /></button
                    >
                </li>
            {/each}
        </ul>
    </aside>
    <div
        class="wrappper flex-1 min-w-0 min-h-0 h-screen"
        bind:clientWidth={viewerWidth}
        bind:clientHeight={viewerHeight}
    >
        <svg
            width={viewerWidth}
            height={viewerHeight}
            class="touch-none"
            on:mouseleave={() =>
                getRecordKeys(isHolding).forEach((node) =>
                    isHolding[node] ? (isHolding[node] = false) : false
                )}
            on:pointermove={handleNodePointerMove}
            on:pointerup={bubbleDownPointerUp}
        >
            <defs>
                <marker
                    id="arrowhead"
                    markerWidth="10"
                    markerHeight="7"
                    refX="10"
                    refY="3.5"
                    orient="auto"
                >
                    <polygon points="0 0, 10 3.5, 0 7" />
                </marker>
            </defs>
            <g transform="translate({viewerWidth / 2} {viewerHeight / 2})">
                {#key edges}
                    {#each edges as edge, i (i)}
                            <GEdge
                                {...getLineBetween(
                                    nodes[edge[0]],
                                    nodes[edge[1]]
                                )}
                                r={nodeRadius}
                            />
                    {/each}
                {/key}
                {#each getRecordKeys(nodes) as node, i (i)}
                        <GNode
                            x={nodes[node][0]}
                            y={nodes[node][1]}
                            r={nodeRadius}
                            name={node}
                            on:pointerdown={(e) => handleNodePointerDown(e, node)}
                            on:pointerup={(e) => handleNodePointerUp(e, node)}
                            on:pointermove
                        />
                {/each}
            </g>
        </svg>
    </div>
</main>

<style lang="postcss">
    :global(html) {
        background-image: linear-gradient(to right, theme(colors.gray.300) 1px, transparent 1px),
            linear-gradient(to bottom, theme(colors.gray.300) 1px, theme(colors.gray.50) 1px);
        background-size: 1.5rem 1.5rem;
    }
    :focus {
        outline: 2px solid theme(colors.cyan.300);
    }
</style>
