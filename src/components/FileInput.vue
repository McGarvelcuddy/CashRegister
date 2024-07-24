<script>
  import { ref } from 'vue'
  import Papa from 'papaparse';
  import { saveAs } from 'file-saver';
  import OptionsSelect from './OptionsSelect.vue'

  const checkedOptions = ref([])
  
  function subtractAndRound(given, due, decimalPlaces = 2) {
    const result = due - given;
    if (result < 0) {
      return 'Negative Result';
    }
    return Number(result.toFixed(decimalPlaces));
  }

  export default {
    components: {
      OptionsSelect
    },
    data() {
      return {
        fileContent: null,
        processedContent: null,
        changeAmount: null,
      };
    },
    methods: {
      getSelectedOptions(newOptions) {
        checkedOptions.value = newOptions
        console.log(checkedOptions);
      },
      handleFileUpload(event) {
        const file = event.target.files[0];
        if (file) {
          const reader = new FileReader();
          reader.onload = (e) => {
            this.fileContent = e.target.result;
            this.processFileContent();
          };
          reader.readAsText(file);
        }
      },
      processFileContent() {
        // Parsing CSV file and converting to JSON
        Papa.parse(this.fileContent, {
          header: false,
          skipEmptyLines: true,
          complete: (results) => {
            this.processedContent = results.data;

            this.changeAmount = results.data.map((set) => subtractAndRound(set[0], set[1]))
            console.log(this.changeAmount)
          }
        });
      },
      downloadFile() {
        const csv = Papa.unparse(this.processedContent);
        const blob = new Blob([csv], { type: 'text/csv;charset=utf-8;' });
        saveAs(blob, 'processed_file.csv');
      },
    },
  };
</script>

<template>
  <div>
    <input type="file" @change="handleFileUpload" />
    <br>
    <OptionsSelect @update:checked-options="getSelectedOptions"/>
    <br>
    <button @click="downloadFile">Download Processed File</button>
  </div>
</template>