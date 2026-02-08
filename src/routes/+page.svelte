<script lang="ts">
    type DataRow = {
        Classification: string;
        Type: string;
        Name: string;
    };
    import Papa from "papaparse";
    let result = $state<DataRow[] | null>(null);
    function handleFileUpload(event: Event) {
        const input = event.target as HTMLInputElement;
        const file = input.files?.[0];
        const papa = Papa.parse(file, {
            header: true,
            skipEmptyLines: true,
        });
        result = papa.data as DataRow[];
    }
</script>

<div class="p-8 max-w-4xl mx-auto">
    <h1 class="text-3xl font-bold mb-6">Blurry Spreadsheet Viewer</h1>
    <!-- File Input -->
    <div
        class="mb-8 p-6 bg-gray-50 rounded-lg border border-dashed border-gray-300 text-center"
    >
        <label for="file" class="block text-lg font-medium mb-2"
            >Upload CSV</label
        >
        <input
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
                    {#each result as column}
                        <th
                            class="border border-gray-300 px-4 py-2 text-left font-semibold"
                        >
                            {column}
                        </th>
                    {/each}
                </tr>
            </thead>
            <tbody>
                {#each result as row}
                    <tr class="hover:bg-gray-50">
                        {#each columns as column}
                            <td class="border border-gray-300 px-4 py-2">
                                {row[column]}
                            </td>
                        {/each}
                    </tr>
                {/each}
            </tbody>
        </table>
    </div>
</div>
