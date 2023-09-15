<script>
// import HelloWorld from './components/HelloWorld.vue'
// import TheWelcome from './components/TheWelcome.vue'

export default {
	name: 'DurationInput',
	props: ['value'],
	data() {
		return {
			timeEnd: null,
			totalTime: 0,
			tapValue: 30 * 1000,
			tapMode: 'additive',
			maxTime: null,
			timeRemaining: 0,
			timeout: null,
			animation: null,
			confirmingReset: false,
			addingTime: false,
			mode: 'menu'
		}
	},
	components: {
		// HelloWorld,
		// TheWelcome,
	},

	methods: {
		changeMinutes(e) {
			const newMinutes = e.currentTarget.value;
			const duration = (newMinutes * 60 * 1000) + ((this.seconds || 0) * 1000);
			this.emitChange(duration);
		},
		changeSeconds(e) {
			const newSeconds = e.currentTarget.value;
			const duration = ((this.minutes || 0) * 60 * 1000) + (newSeconds * 1000);
			this.emitChange(duration);
		},
		emitChange(duration) {
			const value = Math.max(duration, 1000);
			this.$emit('didChange', value)
		}
	},
	computed: {
		minutes() {
			return this.value ? Math.floor(this.value / 1000 / 60) : "";
		},
		seconds() {
			return this.value ? String(Math.floor((this.value / 1000)) % 60).padStart(2, "0") : "";
		}
	}
}

</script>`

<template>
	<div class="input-wrapper">
		<input type="number"
			:value="minutes"
			placeholder="--"
			min="0"
			@input="changeMinutes">
		:
		<input type="number"
			:value="seconds"
			placeholder="--"
			:min="value < 60000 ? 1 : undefined"
			@input="changeSeconds">
	</div>
</template>

<style scoped>
input {
	box-sizing: border-box;
	width: 2em;
	text-align: center;
}
input::-webkit-outer-spin-button,
input::-webkit-inner-spin-button {
    /* display: none; <- Crashes Chrome on hover */
    -webkit-appearance: none;
    margin: 0; /* <-- Apparently some margin are still there even though it's hidden */
}

input[type=number] {
    -moz-appearance:textfield; /* Firefox */
}
</style>
