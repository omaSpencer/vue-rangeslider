<script setup>
// This starter template is using Vue 3 <script setup> SFCs
// Check out https://v3.vuejs.org/api/sfc-script-setup.html#sfc-script-setup
import RangeSlider from './components/RangeSlider.vue';
</script>

<template>
	<img alt="Vue logo" src="./assets/logo.png" />
	<RangeSlider
		:min="0"
		:maxValue="0"
		:step="1"
		:totalSteps="0"
		:percentPerStep="1"
	/>
</template>

<style>
#app {
	font-family: Avenir, Helvetica, Arial, sans-serif;
	-webkit-font-smoothing: antialiased;
	-moz-osx-font-smoothing: grayscale;
	text-align: center;
	color: #2c3e50;
	margin-top: 60px;
	padding: 0 2rem;
}
</style>
