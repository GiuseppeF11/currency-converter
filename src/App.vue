<script>
import AppMain from './components/AppMain.vue';
import axios from 'axios';

export default {
    components: {
        AppMain,
    },
    data() {
        const today = new Date().toISOString().slice(0, 10); //Prendo la data, la trasformo nel formato "AAAA-MM-GG THH:MM:SS.MSZ",  prendo soli i primi 10 caratteri quindi AAAA-MM-GG
        const oneMonthAgo = new Date();                      //Prendo la data odierna, prendo il mese e lo riduco di uno (un mese prima appunto)
        oneMonthAgo.setMonth(oneMonthAgo.getMonth() - 1);
        const oneMonthAgoFormatted = oneMonthAgo.toISOString().slice(0, 10);
        return {
            response: {},
            history_response: {},
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
            ],
            today: today,
            oneMonthAgo: oneMonthAgoFormatted,
        };
    },
    computed: { //La computed property viene ricalcolata solo quando una delle sue dipendenze viene modificata, quindi è molto reattiva
        
        formattedAmount1: {
            // Getter per ottenere il valore formattato
            get() {
                // Restituisce il valore di amount_1 se è un numero, altrimenti restituisce una stringa vuota
                return isNaN(this.amount_1) ? '' : this.amount_1;
            },
            // Setter per impostare il valore di amount_1
            set(value) {
                // Imposta il valore di amount_1 con il valore fornito
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
            //QUERY URL per la chiamata del valore di base
            const currentUrl = 'https://api.frankfurter.app/latest?amount=' +
                this.amount_1 +
                '&from=' + this.currency_1 +
                '&to=' + this.currency_2;

            //QUERY URL per la chiamata della cronologia del tasso nell'ultimo mese
            const historyUrl = `https://api.frankfurter.app/${this.oneMonthAgo}..${this.today}?from=${this.currency_1}&to=${this.currency_2}`;

            axios.all([
                    axios.get(currentUrl),
                    axios.get(historyUrl)
                ])
                .then(axios.spread((currentRes, historyRes) => {
                    this.response = currentRes.data;
                    this.updateAmount2();
                    console.log('Rapporto attuale:', this.response);

                    // Estrai i valori numerici dall'oggetto annidato
                    const valuesArray = Object.values(historyRes.data.rates).map(obj => obj[this.currency_2]);

                    console.log(historyRes.data.rates)

                    this.history_response = historyRes.data.rates

                    // Chiamata chart solo quando i dati sono pronti
                    this.chart();
                }))
                .catch((error) => {
                    console.error('Errore durante le richieste API:', error);
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
        },

        chart() {
            var lastMonth = [];
            var values = [];

            Object.keys(this.history_response).forEach(date => {
                // Trasformo la data nel formato desiderato (giorno mese)
                const formattedDate = formatDate(date);

                // Aggiungo la data formattata alle categorie
                lastMonth.push(formattedDate);

                // Aggiungo il valore della valuta come stringa ai dati della serie
                values.push(String(this.history_response[date][this.currency_2]));
            });

            // Funzione per formattare la data nel formato "giorno mese"
            function formatDate(dateString) {
                // Divide la stringa della data in parti separate da "-"
                const dateParts = dateString.split("-");
                // Estrae l'anno, il mese e il giorno dalle parti della data
                const year = dateParts[0];
                const month = dateParts[1];
                const day = dateParts[2];
                // Crea un nuovo oggetto Data usando l'anno, il mese (sottraendo 1 poiché i mesi in JavaScript sono indicizzati da 0) e il giorno
                const dateObject = new Date(year, month - 1, day);
                // Ottiene il giorno del mese dalla data
                const formattedDay = dateObject.getDate();
                // Ottiene il nome del mese dalla data, utilizzando la formattazione specificata per la lingua italiana
                const formattedMonth = new Intl.DateTimeFormat('it-IT', { month: 'long' }).format(dateObject);
                // Restituisce una stringa nel formato "giorno mese" (ad esempio, "1 gennaio")
                return `${formattedDay} ${formattedMonth}`;
            }

            console.log("Categorie sull'asse x:", lastMonth);
            console.log("Dati della serie:", values);

    var options = {
        series: [{
            name: "Value",
            data: values,
        }],
        chart: {
            height: 350,
            type: 'area',
            zoom: {
                enabled: false
            },
        },
        dataLabels: {
            enabled: false,
        },
        stroke: {
            curve: 'smooth'
        },
        title: {
            text: "Rapporto delle valute nell' ultimo mese",
            align: 'left',
            style: {
                color: '#A9D2B4' // Cambia il colore del titolo qui
            }
        },
        xaxis: {
            categories: lastMonth,
            labels: {
                style: {
                    colors: '#A9D2B4' // Cambia il colore delle etichette sull'asse x qui
                }
            }
        },
        yaxis: {
            labels: {
                style: {
                    colors: '#A9D2B4' // Cambia il colore delle etichette sull'asse y qui
                }
            }
        },
        colors:['#39DA62']
    };

    var chart = new ApexCharts(document.querySelector("#chart"), options);
    chart.render();
    // Esegui un aggiornamento forzato del grafico dopo una breve pausa
    setTimeout(() => {
        window.dispatchEvent(new Event('resize'));
    }, 100);
        },
    },
    watch: {
        currency_1(newValue, oldValue) {
            this.updateRequest();
        },
        currency_2(newValue, oldValue) {
            this.updateRequest();
        }
    },

    mounted() {
        // Al caricamento della pagina effettuo subito una chiamata con i valori di default
        this.updateRequest();
        this.chart();
    }
};
</script>

<template>
    <div class="body">
        <div class="container">
            <AppMain />
            <section>
                <h1 class="text-center primary fw-bold ">CURRENCY CONVERTER</h1>

                <!-- ANTEPRIMA CONVERSIONE -->
                <p class="secondary mb-1" >
                    <span v-if="!isNaN(amount_1)">{{ amount_1 }}</span>  {{ getCurrencySymbol(currency_1) }} è uguale a
                </p>
                <h1 class="secondary"  v-for="(currency, key) in response.rates" :key="key">
                    <span v-if="!isNaN(amount_2)"> {{ amount_2 }} </span>  {{ getCurrencySymbol(currency_2) }}
                </h1>
            </section>
        
            <section class="px-3">
                <!-- VALORE DA CONVERTIRE -->
                <div class="row mb-3">
                    <input class="col block-left" type="text" v-model="formattedAmount1" @input="updateAmount2" placeholder="Inserisci il valore da convertire..." pattern="[0-9]*"> <!-- Ammette valori da 0 a 9 che possono essere ripetuti (*) -->
                    
                    <select class="col block-right" v-model="currency_1" @change="updateRequest">
                        <option value="">Seleziona la valuta di partenza...</option>                        <!-- Disabilito l'opzione se è già presente nell'altra select -->
                        <option v-for="currency in currencies" :value="currency.code" :key="currency.code" :disabled="currency.code === currency_2" :class="{ 'disabled-select': currency.code === currency_2 }"> {{ currency.name }} </option>
                    </select>
                </div>

                <!-- VALORE CONVERTITO -->
                <div class="row">
                    <input class="col block-left" type="text" v-model="formattedAmount2" @input="updateAmount1" placeholder="Inserisci il valore da convertire..." pattern="[0-9]*">
                    
                    <select class="col block-right" v-model="currency_2" @change="updateRequest">
                        <option value="">Seleziona la valuta di destinazione...</option>                    <!-- Disabilito l'opzione se è già presente nell'altra select -->
                        <option class=" hover"v-for="currency in currencies" :value="currency.code" :key="currency.code" :disabled="currency.code === currency_1" :class="{ 'disabled-select': currency.code === currency_1 }"> {{ currency.name }} </option>
                    </select>
                </div>
            </section>

            <section class="mt-5">

                <div id="chart">
                </div>
            </section>
        </div>
    </div>
</template>

<style lang="scss">
@use "assets/scss/main" as *;
@import "assets/scss/partials/reset";

//COLORS
.primary {
    color: #39DA62;
}

.secondary {
    color: #A9D2B4;
}

.tertiary {
    color: #343434;
}


//LAYOUT
.body {
    background-color: #343434;
    min-height: 100vh;
    display: flex;
    font-family: "Roboto Condensed", sans-serif;
    padding: 10px 0;
}

.container {
    border: 3px solid #39DA62;
    border-radius: 10px;
    padding: 20px;
    max-width: 50vw;
    display: flex;
    flex-direction: column;
    justify-content: space-between;
    height: 95vh;
}


//ELEMENTS
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
    cursor: pointer;
}

.block-left , .block-right {
    &:hover {
        background-color: #88cd9a;
    }
}

.disabled-select {
    background-color: #b6cabb;
    color: #f1f1f1;
}

#chart {
  width: 100%;
}


</style>
