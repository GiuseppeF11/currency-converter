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
            amount_1: '1.00', // Assegno '1' come valore predefinito per l'input
            amount_2: '',
            currency_1: 'EUR', // Assegno EUR come valore predefinito per la select di partenza
            currency_2: 'USD', // Assegno USD come valore predefinito per la select di destinazione
            currencies: [
                { code: 'EUR', name: 'Euro', symbol: '€' },
                { code: 'BGN', name: 'Bulgarian Lev', symbol: 'лв' },
                { code: 'BRL', name: 'Brazilian Real', symbol: 'R$' },
                { code: 'CAD', name: 'Canadian Dollar', symbol: 'CA$' },
                { code: 'CHF', name: 'Swiss Franc', symbol: 'CHF' },
                { code: 'CNY', name: 'Chinese Yuan', symbol: 'CN¥' },
                { code: 'CZK', name: 'Czech Koruna', symbol: 'Kč' },
                { code: 'DKK', name: 'Danish Krone', symbol: 'kr' },
                { code: 'GBP', name: 'British Pound Sterling', symbol: '£' },
                { code: 'HKD', name: 'Hong Kong Dollar', symbol: 'HK$' },
                { code: 'HUF', name: 'Hungarian Forint', symbol: 'Ft' },
                { code: 'IDR', name: 'Indonesian Rupiah', symbol: 'Rp' },
                { code: 'ILS', name: 'Israeli New Sheqel', symbol: '₪' },
                { code: 'INR', name: 'Indian Rupee', symbol: '₹' },
                { code: 'ISK', name: 'Icelandic Króna', symbol: 'kr' },
                { code: 'JPY', name: 'Japanese Yen', symbol: '¥' },
                { code: 'KRW', name: 'South Korean Won', symbol: '₩' },
                { code: 'MXN', name: 'Mexican Peso', symbol: 'MX$' },
                { code: 'MYR', name: 'Malaysian Ringgit', symbol: 'RM' },
                { code: 'NOK', name: 'Norwegian Krone', symbol: 'kr' },
                { code: 'NZD', name: 'New Zealand Dollar', symbol: 'NZ$' },
                { code: 'PHP', name: 'Philippine Peso', symbol: '₱' },
                { code: 'PLN', name: 'Polish Zloty', symbol: 'zł' },
                { code: 'RON', name: 'Romanian Leu', symbol: 'lei' },
                { code: 'SEK', name: 'Swedish Krona', symbol: 'kr' },
                { code: 'SGD', name: 'Singapore Dollar', symbol: 'S$' },
                { code: 'THB', name: 'Thai Baht', symbol: '฿' },
                { code: 'TRY', name: 'Turkish Lira', symbol: '₺' },
                { code: 'USD', name: 'United States Dollar', symbol: '$' },
                { code: 'ZAR', name: 'South African Rand', symbol: 'R' }
            ]

        };
    },
    computed: { //La computed property viene ricalcolata solo quando una delle sue dipendenze viene modificata, quindi è molto reattiva
        formattedAmount1: {
            get() {
                return isNaN(this.amount_1) ? '' : this.amount_1;
            },
            set(value) {
                this.amount_1 = value;
            }
        },
        formattedAmount2: {
            get() {
                return isNaN(this.amount_2) ? '' : this.amount_2;
            },
            set(value) {
                this.amount_2 = value;
            }
        }
    },

    methods: {
        //Definisco la funzione per aggiornare la conversione ad ogni digitazione
        updateRequest() {
            const url = 'https://api.frankfurter.app/latest?amount=' + this.amount_1 + //Dato un valore
                '&from=' + this.currency_1 + //Data la valuta da convertire
                '&to=' + this.currency_2;    //Data la valuta convertita
            axios
                .get(url)   //effettuo la chiamata API tramite query
                .then((res) => {
                    this.response = res.data;
                    this.updateAmount2(); // Aggiorna amount_2 quando c'è una nuova risposta
                    console.log(this.response);
                })
                .catch((error) => { //
                    console.error('Errore durante la richiesta API:', error);
                });
        },

        updateAmount2() {
            if (this.currency_1 !== this.currency_2) { // Verifico che le valute di partenza e destinazione siano diverse
                const rate = this.response.rates[this.currency_2]; // Ottengo il tasso di conversione dalla risposta dell'API
                this.amount_2 = (parseFloat(this.amount_1) * rate).toFixed(2); // Calcolo il valore convertito utilizzando il tasso e l'importo di partenza e lo arrotondo a 2 decimali
            } else {
                this.amount_2 = this.amount_1; // Se le valute sono uguali, mantengo lo stesso valore
            }
        },

        updateAmount1() {
            if (this.currency_1 !== this.currency_2) {
                const rate = this.response.rates[this.currency_2];
                this.amount_1 = (parseFloat(this.amount_2) / rate).toFixed(2);
            } else {
                this.amount_1 = this.amount_2;
            }
        },

        getCurrencySymbol(currencyCode) {
            const currency = this.currencies.find(curr => curr.code === currencyCode); // Cerco la valuta corrispondente al codice della valuta specificato
            return currency ? currency.symbol : '';  // Trovata la valuta, restituisci il simbolo della valuta, altrimenti restituisci una stringa vuota
        }
    },
    mounted() {
        // Al caricamento della pagina effettuo subito una chiamata con i valori di default
        this.updateRequest();
    }
};
</script>

<template>
    <div class="body">
        <div class="container">
            <AppMain />
            <h1 class="text-center primary fw-bold ">CURRENCY CONVERTER</h1>
            <p class="secondary mb-1" >
                <span v-if="!isNaN(amount_1)">{{ amount_1 }}</span>  {{ getCurrencySymbol(currency_1) }} è uguale a
            </p>
            <h1 class="secondary"  v-for="(currency, key) in response.rates" :key="key">
                <span v-if="!isNaN(amount_2)"> {{ amount_2 }} </span>  {{ getCurrencySymbol(currency_2) }}
            </h1>
        
            <div class="row mb-3">
                <input class="col block-left" type="text" v-model="formattedAmount1" @input="updateAmount2" placeholder="Inserisci il valore da convertire..." pattern="[0-9]*"> <!-- Ammette valori da 0 a 9 che possono essere ripetuti (*) -->
                
                <select class="col block-right" v-model="currency_1" @change="updateRequest">
                    <option value="">Seleziona la valuta di partenza...</option>                        <!-- Disabilito l'opzione se è già presente nell'altra select -->
                    <option v-for="currency in currencies" :value="currency.code" :key="currency.code" :disabled="currency.code === currency_2" :class="{ 'disabled-select': currency.code === currency_2 }"> {{ currency.name }} </option>
                </select>
                
            </div>

            <div class="row">
                <input class="col block-left" type="text" v-model="formattedAmount2" @input="updateAmount1" placeholder="Inserisci il valore da convertire..." pattern="[0-9]*">
                
                <select class="col block-right" v-model="currency_2" @change="updateRequest">
                    <option value="">Seleziona la valuta di destinazione...</option>                    <!-- Disabilito l'opzione se è già presente nell'altra select -->
                    <option v-for="currency in currencies" :value="currency.code" :key="currency.code" :disabled="currency.code === currency_1" :class="{ 'disabled-select': currency.code === currency_1 }"> {{ currency.name }} </option>
                </select>
                
            </div>
        </div>
    </div>
</template>

<style lang="scss">
@use "assets/scss/main" as *;
@import "assets/scss/partials/reset";

//COLOR
.primary {
    color: #39DA62;
}

.secondary {
    color: #A9D2B4;
}

.tertiary {
    color: #343434;
}

.body {
    background-color: #343434;
    min-height: 100vh;
    padding: 5rem 2rem;
    font-family: "Roboto Condensed", sans-serif;
}

.container {
    border: 3px solid #39DA62;
    border-radius: 10px;
    padding: 20px;
    max-width: 50vw;
}

.block-left {
    background-color: #A9D2B4;
    border: none;
    border-bottom: 2px solid #39DA62;
    border-top-left-radius: 5px;
    border-bottom-left-radius: 5px;
    text-align: center;
    font-size: 15px;
    min-height: 45px;
    font-weight: 600;
}

.block-right {
    background-color: #A9D2B4;
    border: none;
    border-bottom: 2px solid #39DA62;
    border-top-right-radius: 5px;
    border-bottom-right-radius: 5px;
    text-align: center;
    font-size: 15px;
    min-height: 45px;
    font-weight: 600;
}


.disabled-select {
    background-color: #b6cabb;
    color: #999;
}

</style>
