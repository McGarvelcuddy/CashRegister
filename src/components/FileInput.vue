<script lang="ts">
  import {defineComponent} from 'vue'
  import Papa from 'papaparse'
  import { saveAs } from 'file-saver'
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
      "pluralName": "pennies",
      "value": 0.01
    }
  ]
  
  function subtractAndRound(given: number, due: number) {
    const result = due - given
    if (result < 0) {
      return 'Negative Result'
    }
    return Number(result.toFixed(2))
  }

  function getCurrencyBreakdown(
    amount: number|string, randomize: boolean = false, currency: Currency[] = defaultCurrency
  ): { [key: string]: number} {
    const breakdown: { [key: string]: number} = {}

    if (randomize) {
      currency = currency.slice().sort(() => Math.random() - 0.5)
    } else {
      currency = currency.slice().sort((a: Currency, b: Currency) => b.value - a.value)
    }
    if(typeof amount === 'number') {
      for (const coin of currency) {
        const count = Math.floor(Number(amount.toFixed(2)) / coin.value)
        if (count > 0) {
          const name = count > 1 ? coin.pluralName : coin.singularName
          breakdown[name] = count
          amount -= count * coin.value
        }
      }
    }

    return breakdown
  }

  export default defineComponent({
    name: "file-input",
    components: {
      OptionsSelect
    },
    data() {
      return {
        changeFileContent: null as string | ArrayBuffer | null,
        localeFileContent: null as string | ArrayBuffer | null,
        processedContent: [] as any[],
        changeAmount: [] as (number|string)[],
        checkedOptions: [] as string[],
        divisor: 3 as number,
        currentResults: [] as { [key: string]: number}[],
        currency: defaultCurrency as Currency[]
      }
    },
    methods: {
      getSelectedOptions(newOptions: string[]) {
        this.checkedOptions = newOptions
      },
      setDivisor(newDivisor: number) {
        this.divisor = newDivisor
      },
      handleExchangeFileUpload(event: Event) {
        const input = event.target as HTMLInputElement
        const file = input.files ? input.files[0] : null
        if (file) {
          const reader = new FileReader()
          reader.onload = (e) => {
            try {
              this.changeFileContent = e.target?.result || null
              this.processFileContent()
            } catch (error) {
              console.error("Error parsing file:", error)
            }
            
          }
          reader.readAsText(file)
        }
      },
      handleLocaleFileUpload(event: Event) {
        const input = event.target as HTMLInputElement
        if (input.files && input.files[0]) {
          const file = input.files[0]
          const reader = new FileReader()
          reader.onload = (e) => {
            try {
              const json = e.target?.result as string
              this.currency = JSON.parse(json)
            } catch (error) {
              console.error("Error parsing JSON:", error)
            }
          }
          reader.readAsText(file)
        }
      },
      processFileContent() {
        // Parsing CSV file and converting to JSON
        Papa.parse(this.changeFileContent as string, {
          header: false,
          skipEmptyLines: true,
          complete: (results) => {
            this.processedContent = results.data as any[]

            this.changeAmount = this.processedContent.map((set: any[]) => subtractAndRound(set[0], set[1]) as number)
          }
        })
      },
      downloadFile() {
        this.calculateCurrency()
        const formattedResults = this.currentResults.map((entry) => {
          const parts = Object.entries(entry).map(([key, value]) => `${value} ${key}`)
          return parts.join(',')
        }).join('\n\n')

        const blob = new Blob([formattedResults], { type: 'text/csvcharset=utf-8' })
        saveAs(blob, 'processed_file.csv')
      },
      calculateCurrency() {
        this.currentResults = this.changeAmount.map((value, index) => getCurrencyBreakdown(value, ((this.processedContent[index][0] * 100) % this.divisor === 0 && this.checkedOptions.includes("÷x")), this.currency))
      }
    },
  }
)
</script>

<template>
  <div :style="{display: 'grid',  justifyItems: 'stretch', rowGap: '1rem'}">
    <div class="upload-box">
      <label for="languageUploadButton">Currency Locale To Be Processed (default US) >  </label>
      <input id="languageUploadButton" type="file" @change="handleLocaleFileUpload" />
    </div>
    <div class="upload-box">
      <label for="exchangeFileUploadButton">File To Be Processed >  </label>
      <input id="exchangeFileUploadButton" type="file" @change="handleExchangeFileUpload" />
    </div>

    <OptionsSelect @update:checked-options="getSelectedOptions" @update:divisible-by="setDivisor"/>

    <button @click="downloadFile" :disabled="changeAmount.length <= 0">Download Processed File</button>
  </div>
</template>

<style scoped>
.upload-box {
  border: solid;
  padding: 1rem;
}
</style>