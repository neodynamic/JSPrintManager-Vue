<template>
    <div id="installedPrinters">
        <label for="installedPrinterName">Select an installed Printer: </label>
        <select name="printer" id="printer" v-model="selected_printer">
            <option disabled n-bind:key="undefined">Select printer</option>
            <option v-for="printer in printer_list" v-bind:key="printer">{{ printer }}</option>
        </select>
    </div>
</template>
<script>
export default {
    name: "Printers",
    props: ['printer_list'],
    data: function () {        
        return {
            selected_printer: ''
        }
    },
    watch: {
        selected_printer(value) {
            this.$emit('change', value);
        }
    }
}
</script>