<script>
import AppMain from './components/AppMain.vue';
import axios from 'axios';

export default {
    components: {
        AppMain,
    },  
    data() {
        return {
            response: {},
            amount_1: '1', // Imposta '1' come valore predefinito per l'input
            amount_2: '',
            value_1: 'EUR', // Imposta EUR come valore predefinito per la select di partenza
            value_2: 'USD', // Imposta USD come valore predefinito per la select di destinazione
            currencies: ['EUR', 'BGN', 'BRL', 'CAD', 'CHF', 'CNY', 'CZK', 'DKK', 'GBP', 'HKD', 'HUF', 'IDR', 'ILS', 'INR', 'ISK', 'JPY', 'KRW', 'MXN', 'MYR', 'NOK', 'NZD', 'PHP', 'PLN', 'RON', 'SEK', 'SGD', 'THB', 'TRY', 'USD', 'ZAR']
        };
    },
    methods: {
        updateRequest() {
            const url = 'https://api.frankfurter.app/latest?amount=' + this.amount_1 +
                        '&from=' + this.value_1 + 
                        '&to=' + this.value_2;
            axios
                .get(url)
                .then((res) => {
                    this.response = res.data;
                    console.log(this.response);
                })
                .catch((error) => {
                    console.error('Errore durante la richiesta API:', error);
                });
        },
    },
    mounted() {
        // Chiamata iniziale al caricamento della pagina
        this.updateRequest();
    }
};
</script>

<template>
    <div>
        <AppMain />
        <h1 class="text-center">Value Converter</h1>
        <h1>from {{ response.base }} - {{ response.amount }} to 
            <span v-for="(value, key) in response.rates" :key="key">
                {{ key }} - {{ value }}
            </span>
        </h1>
        
        <input type="text" v-model="amount_1" @input="updateRequest" placeholder="Inserisci il valore da convertire...">
        
        <select v-model="value_1" @change="updateRequest">
            <option value="">Seleziona la valuta di partenza...</option>
            <option v-for="currency in currencies" :value="currency" :key="currency">{{ currency }}</option>
        </select>
        
        <select v-model="value_2" @change="updateRequest">
            <option value="">Seleziona la valuta di destinazione...</option>
            <option v-for="currency in currencies" :value="currency" :key="currency">{{ currency }}</option>
        </select>
    </div>
</template>

<style lang="scss">
@use "assets/scss/main" as *;
@import "assets/scss/partials/reset";
</style>
