<template>
    <div id="app">
        <canv></canv>
    </div>
</template>

<script>
import Canv from './components/Canv.vue'

export default {
    name: 'app',

    components: {
        Canv
    }
}
</script>

<style>
#app {
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
}
</style>