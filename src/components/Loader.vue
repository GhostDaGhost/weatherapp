<script lang="ts">
    export default {
        name: 'Loader',
        data() {
            return {
                showLoader: false
            }
        },
        methods: {
            toggleLoader (newLoaderState: boolean = false): void {
                this.showLoader = newLoaderState;
            }
        }
    }
</script>

<template>
    <div class="spinner_container">
        <svg class="spinner" viewBox="0 0 100 100" xmlns="http://www.w3.org/2000/svg">
            <circle class="circle" cx="50" cy="50" r="45"/>
        </svg>
    </div>
</template>

<style lang="scss" scoped>
    @import '../styles/Loader.scss';
</style>
