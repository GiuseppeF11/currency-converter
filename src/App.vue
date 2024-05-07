<script>
import AppMain from './components/AppMain.vue';
import axios from 'axios';

export default {
    data() {
        return {
            response : '',
            content : '',
            value_1: 'EUR',
            value_2: 'BRL',
        };
    },
    components: {
        AppMain,
    },  
    methods: {

    },
    mounted() {
        const url = 'https://api.frankfurter.app/latest?from=' + this.value_1 + '&to=' + this.value_2;
        axios
            .get(url)
            .then((res) => {
                this.response = res.data
                console.log(this.response)
            });
    }
}
</script>

<template>

    <AppMain />

    <h1 class="text-center">Value Converter</h1>
    
        <h1>{{ response.base }} - {{ response.amount }}</h1>
        <ul>
            <li v-for="(value, key) in response.rates">
                {{ key }} - {{ value }}
            </li>
        </ul>

        <input type ="text" v-model="content" placeholder = "Inserisci qualcosa">

</template>

<style lang="scss">
@use "assets/scss/main" as *;
@import "assets/scss/partials/reset";
</style>
