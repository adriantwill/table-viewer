<script lang="ts">
    import Papa, { type ParseResult } from "papaparse";

    let result = $state<Record<string, any>[]>([]);

    // Extract keys dynamically from the first row
    let headers = $derived(result.length > 0 ? Object.keys(result[0]) : []);

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
            },
        });
    }
</script>

<div class="p-8 max-w-4xl mx-auto">
    <h1 class="text-3xl font-bold mb-6">Table Viewer</h1>
    <!-- File Input -->
    <div
        class="mb-8 p-6 bg-gray-50 rounded-lg border border-dashed border-gray-300 text-center"
    >
        <label for="file" class="block text-lg font-medium mb-2"
            >Upload CSV</label
        >
        <input
            id="file"
            type="file"
            accept=".csv"
            onchange={handleFileUpload}
            class="mx-auto"
        />
    </div>
    <!-- The Table -->
    <div class="overflow-x-auto">
        <table class="min-w-full border-collapse border border-gray-300">
            <thead>
                <tr class="bg-gray-100">
                    {#each headers as header}
                        <th
                            class="border border-gray-300 px-4 py-2 text-left font-semibold"
                        >
                            {header}
                        </th>
                    {/each}
                </tr>
            </thead>
            <tbody>
                {#each result as row, i}
                    <tr class="hover:bg-gray-50">
                        {#each headers as header, j}
                            {#if rowSpans[i]?.[j] > 0}
                                <td
                                    class="border border-gray-300 px-4 py-2 {j > 0 ? 'blur-sm' : ''} cursor-pointer transition-all duration-200"
                                    rowspan={rowSpans[i][j] > 1
                                        ? rowSpans[i][j]
                                        : undefined}
                                    onclick={(e) => j > 0 && e.currentTarget.classList.remove('blur-sm')}
                                >
                                    {row[header]}
                                </td>
                            {/if}
                        {/each}
                    </tr>
                {/each}
            </tbody>
        </table>
    </div>
</div>
