<template>
    <a class="btn btn-primary" @click="downloadCSV">Download as CSV</a>
</template>

<script>
export default {
    name: 'Downloader',
    props: {
        selections: {
            type: Array,
            required: true
        }
    },
    methods: {
        downloadCSV() {
            if (!this.selections.length) return;

            // Convert selection to CSV
            const keys = ['name', 'color', 'percentage', 'area'];
            const header = ['name', 'color (hex)', 'percentage', 'squaremeter'];
            const csvRows = [
                ['name', 'color (hex)', 'percentage', 'squaremeter'].join(','), // header
                ...this.selections.map(item =>
                    [
                        `"${item.name ? String(item.name).replace(/"/g, '""') : (typeof item.color === 'object' && item.color !== null && 'hex' in item.color ? item.color.hex : String(item.color))}"`,
                        `"${typeof item.color === 'object' && item.color !== null && 'hex' in item.color ? item.color.hex : String(item.color)}"`,
                        `"${String(item.percentage).replace(/"/g, '""')}"`,
                        `"${String(item.area).replace(/"/g, '""')}"`
                    ].join(',')
                )
            ];
            const csvContent = csvRows.join('\n');

            // Create a blob and trigger download
            const blob = new Blob([csvContent], { type: 'text/csv;charset=utf-8;' });
            const link = document.createElement('a');
            link.href = URL.createObjectURL(blob);
            const dateStr = new Date().toISOString().slice(0,10); // YYYY-MM-DD
            link.setAttribute('download', `selection_${dateStr}.csv`);
            document.body.appendChild(link);
            link.click();
            document.body.removeChild(link);
            URL.revokeObjectURL(link.href);
        }
    }
}
</script>