<script>
import AppMain from './components/AppMain.vue';
import axios from 'axios';

export default {
    data() {
        return {

        };
    },
    components: {
        AppMain,
    },  
    methods: {

    },
    mounted() {
        axios
            .get('')
            .then((response) => {
                console.log(response)
            });
    }
}
</script>

<template>

    <AppMain />
</template>

<style lang="scss">
@use "assets/scss/main" as *;
@import "assets/scss/partials/reset";
</style>
