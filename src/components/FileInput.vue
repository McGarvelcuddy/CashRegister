<script lang="ts">
  import Vue from 'vue'
  import Papa from 'papaparse';
  import { saveAs } from 'file-saver';
  import OptionsSelect from './OptionsSelect.vue'
  import Currency from './../types/Currency'

  const defaultCurrency: Currency[] = [
    {
      "singularName": "dollar",
      "pluralName": "dollars",
      "value": 1
    },
    {
      "singularName": "quarter",
      "pluralName": "quarters",
      "value": 0.25
    },
    {
      "singularName": "dime",
      "pluralName": "dimes",
      "value": 0.10
    },
    {
      "singularName": "nickle",
      "pluralName": "nickle",
      "value": 0.05
    },
    {
      "singularName": "penny",
      "pluralName": "penny",
      "value": 0.01
    }
  ]
  
  function subtractAndRound(given: number, due: number, decimalPlaces: number = 2) {
    const result = due - given;
    if (result < 0) {
      return 'Negative Result';
    }
    return Number(result.toFixed(decimalPlaces));
  }

  function getCurrencyBreakdown(
    amount: number, currency: Currency[] = defaultCurrency, randomize: boolean = false
  ): { [key: string]: number} {
    const breakdown: { [key: string]: number} = {};

    if (randomize) {
      currency = currency.slice().sort(() => Math.random() - 0.5);
    } else {
      currency = currency.slice().sort((a: Currency, b: Currency) => b.value - a.value);
    }

    for (const coin of currency) {
      const count = Math.floor(amount / coin.value);
      if (count > 0) {
        const name = count > 1 ? coin.pluralName : coin.singularName;
        breakdown[name] = count;
        amount -= count * coin.value;
      }
    }

    return breakdown
  }

  export default Vue.defineComponent({
    components: {
      OptionsSelect
    },
    data() {
      return {
        fileContent: null as string | ArrayBuffer | null,
        processedContent: [] as any[],
        changeAmount: [] as number[],
        checkedOptions: [] as string[],
      };
    },
    methods: {
      getSelectedOptions(newOptions: string[]) {
        this.checkedOptions = newOptions
        console.log(this.checkedOptions);
      },
      handleFileUpload(event: Event) {
        const input = event.target as HTMLInputElement;
        const file = input.files ? input.files[0] : null;
        if (file) {
          const reader = new FileReader();
          reader.onload = (e) => {
            this.fileContent = e.target?.result || null;
            this.processFileContent();
          };
          reader.readAsText(file);
        }
      },
      processFileContent() {
        // Parsing CSV file and converting to JSON
        Papa.parse(this.fileContent as string, {
          header: false,
          skipEmptyLines: true,
          complete: (results) => {
            this.processedContent = results.data as any[];

            this.changeAmount = this.processedContent.map((set: any[]) => subtractAndRound(set[0], set[1]) as number);
            console.log(this.changeAmount);
          }
        });
      },
      downloadFile() {
        const csv = Papa.unparse(this.processedContent);
        const blob = new Blob([csv], { type: 'text/csv;charset=utf-8;' });
        saveAs(blob, 'processed_file.csv');
      },
    },
  }
)
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