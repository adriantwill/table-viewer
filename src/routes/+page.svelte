<script lang="ts">
    import Papa, { type ParseResult } from "papaparse";
    import Pencil from "lucide-svelte/icons/pencil";

    let result = $state<Record<string, any>[]>([]);

    // Extract keys dynamically from the first row
    let headers = $derived(result.length > 0 ? Object.keys(result[0]) : []);
    let revealed = $state(new Set<string>());
    let editingCell = $state<string | null>(null);

    function toggleReveal(i: number, j: number) {
        const key = `${i}-${j}`;
        const newRevealed = new Set(revealed);
        if (newRevealed.has(key)) {
            newRevealed.delete(key);
        } else {
            newRevealed.add(key);
        }
        revealed = newRevealed;
    }
    // Calculate rowspans for cells with empty cells below them
    let rowSpans = $derived(calculateRowSpans(result, headers));

    function calculateRowSpans(
        data: Record<string, any>[],
        cols: string[],
    ): number[][] {
        const spans: number[][] = [];
        for (let i = 0; i < data.length; i++) {
            spans[i] = [];
            for (let col = 0; col < cols.length; col++) {
                const value = data[i][cols[col]];
                if (!value) {
                    spans[i][col] = 0;
                } else {
                    let count = 1;
                    while (
                        i + count < data.length &&
                        !data[i + count][cols[col]]
                    ) {
                        count++;
                    }
                    spans[i][col] = count;
                }
            }
        }
        return spans;
    }
    function handleFileUpload(event: Event) {
        const input = event.target as HTMLInputElement;
        const file = input.files?.[0];

        if (!file) return;

        Papa.parse(file, {
            header: true,
            skipEmptyLines: true,
            complete: (papa: ParseResult<Record<string, any>>) => {
                result = papa.data;
                const newRevealed = new Set<string>();
                for (let i = 0; i < result.length; i++) {
                    newRevealed.add(`${i}-${0}`);
                }
                revealed = newRevealed;
                console.log(result);
            },
        });
    }
</script>

<div class="p-8 max-w-4xl mx-auto bg-surface text-text-main min-h-screen">
    <h1 class="text-3xl font-bold mb-6">Table Viewer</h1>
    <!-- File Input -->
    <div
        class="mb-8 p-6 bg-surface-alt rounded-lg border border-dashed border-border-default text-center"
    >
        <label for="file" class="block text-lg font-medium mb-2 text-text-main"
            >Upload CSV</label
        >
        <input
            id="file"
            type="file"
            accept=".csv"
            onchange={handleFileUpload}
            class="mx-auto text-text-muted file:mr-4 file:py-2 file:px-4 file:rounded-full file:border-0 file:text-sm file:font-semibold file:bg-surface-header file:text-text-main hover:file:bg-border-default cursor-pointer"
        />
    </div>
    <!-- The Table -->
    <div class="overflow-x-auto">
        <table class="min-w-full border-collapse border border-border-default">
            <thead>
                <tr class="bg-surface-header">
                    {#each headers as header}
                        <th
                            class="border border-border-default px-4 py-2 text-left font-semibold"
                        >
                            {header}
                        </th>
                    {/each}
                </tr>
            </thead>
            <tbody>
                {#each result as row, i}
                    <tr class="">
                        {#each headers as header, j}
                            {#if rowSpans[i]?.[j] > 0}
                                <td
                                    class="relative hover:bg-surface-alt border border-border-default px-4 py-2 transition-all duration-200 {!revealed.has(
                                        `${i}-${j}`,
                                    )
                                        ? 'blur-sm'
                                        : ''}"
                                    rowspan={rowSpans[i][j] > 1
                                        ? rowSpans[i][j]
                                        : undefined}
                                    onclick={() => toggleReveal(i, j)}
                                >
                                    {#if revealed.has(`${i}-${j}`)}
                                        <button
                                            class="absolute top-1 right-1 z-10"
                                            onclick={(e) => {
                                                e.stopPropagation();
                                                editingCell =
                                                    editingCell === `${i}-${j}`
                                                        ? null
                                                        : `${i}-${j}`;
                                            }}
                                        >
                                            <Pencil
                                                size={14}
                                                class="text-gray-400"
                                            />
                                        </button>
                                    {/if}
                                    {#if editingCell === `${i}-${j}`}
                                        <input
                                            onclick={(e) => {
                                                e.stopPropagation();
                                            }}
                                            type="text"
                                            bind:value={row[header]}
                                            class="w-full bg-transparent text-text-main focus:outline-none"
                                            autofocus
                                            onblur={() => (editingCell = null)}
                                            onkeydown={(e) => {
                                                if (e.key === "Enter") {
                                                    editingCell = null;
                                                }
                                            }}
                                        />
                                    {:else}
                                        {row[header]}
                                    {/if}
                                </td>
                            {/if}
                        {/each}
                    </tr>
                {/each}
            </tbody>
        </table>
    </div>
</div>
