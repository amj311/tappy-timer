<script>
// import HelloWorld from './components/HelloWorld.vue'
// import TheWelcome from './components/TheWelcome.vue'

export default {
	name: 'App',
	data() {
		return {
			timeEnd: null,
			totalTime: 0,
			tapValue: 30 * 1000,
			timeRemaining: 0,
			timeout: null,
			animation: null,
			confirmingReset: false,
			mode: 'menu'
		}
	},
	components: {
		// HelloWorld,
		// TheWelcome,
	},
	beforeMount() {
		this.animate();
	},
	beforeUnmount() {
		this.reset();
	},

	methods: {
		start() {
			this.mode = 'timer';
			this.addTime();
		},
		onClickReset() {
			if (this.confirmingReset) {
				this.reset();
				this.confirmingReset = false;
			}
			else {
				this.confirmingReset = true;
				setTimeout(() => {
					this.confirmingReset = false;
				}, 1000);
			}
		},
		reset() {
			this.mode = 'menu';
			this.timeEnd = null;
			this.timeRemaining = 0;
			this.totalTime = 0;
			clearInterval(this.timeout);
			cancelAnimationFrame(this.animate);
		},
		addTime() {
			clearTimeout(this.timeout);

			this.timeEnd = (this.timeEnd || Date.now()) + this.tapValue;
			this.totalTime = this.timeEnd - Date.now();
			this.timeout = setTimeout(this.onTimeEnd, this.timeEnd - Date.now());
		},
		onTimeEnd() {
			this.timeEnd = null;
			console.log('end');
			this.playAlarm();
		},
		playAlarm() {
			const audioContext = new AudioContext();
			const oscillator = audioContext.createOscillator();
			oscillator.type = 'sine';
			oscillator.frequency.setValueAtTime(440, audioContext.currentTime);

			oscillator.connect(audioContext.destination);
			oscillator.start();
			setTimeout(() => {
				oscillator.stop();
			}, 2000); // Stop the alarm sound after 2 seconds
		},
		animate() {
			this.updateTimer();
			this.animation = requestAnimationFrame(this.animate);
		},
		updateTimer() {
			if (!this.timeEnd) {
				this.timeRemaining = 0;
				return;
			};
			this.timeRemaining = this.timeEnd - Date.now();
		},
		fmtMSS(milli) {
			let duration = new Date(milli);
			const minutes = duration.getUTCMinutes();
			const seconds = duration.getUTCSeconds();
			const milliseconds = duration.getUTCMilliseconds();
			return `${minutes.toString().padStart(2, "0")}:${seconds.toString().padStart(2, "0")}`;
		},
	},
	computed: {
		remainingRatio() {
			if (!this.timeRemaining) {
				return 0;
			}
			return (this.timeRemaining / this.totalTime)
		},
		timerColor() {
			if (!this.timeRemaining) {
				return 'black';
			}
			return this.timeRemaining <= 11000 ? 'red' : 'green'
		}
	}
}

</script>`

<template>
	<div class="timer-container">
		<div class="title"
			v-if="mode === 'menu'"
		>
			TAPPY TIMER
		</div>
		<div class="start-button"
			v-if="mode === 'menu'"
			@click="start"
			:style="{
				filter: `drop-shadow(0 0 50px green)`
			}"
		>
			Start
		</div>
		<details class="settings"
			v-if="mode === 'menu'"
		>
			<summary>Settings</summary>
			<label>Tap Value: <input type="number" v-model="tapValue"></label>
		</details>
		<div class="reset"
			v-if="mode === 'timer'"
			:class="{ confirming: confirmingReset }"
			@click.stop="onClickReset">
			<span class="icon fa fa-times"></span>
			<span class="confirm">&nbsp;&nbsp;Press again to close</span>
		</div>
		<Transition>
			<div class="svg-wrapper"
				v-if="mode === 'timer'"
				@click="addTime"
				onmousedown="this.classList.add('active')"
				onmouseup="this.classList.remove('active')">

				<svg viewBox="0 0 200 200"
					preserve-aspect-ratio
					:style="{
						filter: `drop-shadow(0 0 50px ${timerColor})`
					}">
					<circle cx="100"
						cy="100"
						r="80"
						fill="#181818" />
					<circle cx="100"
						cy="100"
						r="80"
						fill="none"
						stroke="#555"
						stroke-width="10" />
					<circle cx="100"
						cy="100"
						r="80"
						fill="none"
						:stroke="timerColor"
						stroke-width="10"
						:stroke-dasharray="2 * Math.PI * 80"
						:stroke-dashoffset="2 * Math.PI * 80 * (1 - remainingRatio)"
						transform="rotate(-90 100 100)" />
					<text x="50%"
						y="50%"
						dominant-baseline="middle"
						text-anchor="middle"
						font-size="24"
						fill="#ffffff">
						{{ fmtMSS(timeRemaining) }}
					</text>
				</svg>
			</div>
		</Transition>
	</div>
</template>

<style scoped>
.timer-container {
	position: relative;
	width: 100vw;
	height: 100vh;
	user-select: none;
}

.title {
	position: absolute;
	top: 3em;
	left: 50%;
	transform: translateX(-50%);
	font-size: 3em;
	font-weight: bold;
    width: 100%;
    text-align: center;
}

.start-button {
	position: absolute;
	top: 50%;
	left: 50%;
	transform: translate(-50%, -50%);
    text-align: center;
	background-color: green;
	color: white;
	padding: .5em 1.2em;
	font-weight: bold;
	border-radius: 2em;
	font-size: 2em;
}

.settings {
	position: absolute;
	bottom: 3rem;
	left: 50%;
	transform: translateX(-50%);
    text-align: center;
	width: 100%;
	transition: 500ms;
}

.settings[open] {
	top: 0;
	bottom: 0;
	left: 0;
	right: 0;
	transform: none;
    background: var(--color-background);
}

.settings summary {
	font-size: 1.5em;
}

.svg-wrapper {
	position: absolute;
	top: 0;
	bottom: 0;
	left: 0;
	right: 0;
	display: flex;
	place-items: center;
	justify-content: center;
	flex-wrap: wrap;
	flex-direction: column-reverse;
	padding: 15%;
}

.svg-wrapper svg {
	transition: 250ms;
}

.svg-wrapper.active svg {
	transform: scale(1.1);
}

.reset {
	position: absolute;
	bottom: 1em;
	left: 50%;
	transform: translateX(-50%);
	z-index: 2;
	font-size: 1.5em;
	padding: 1em;
	cursor: pointer;
	display: flex;
	align-items: center;
}

.reset .icon {
	font-size: 1.5em;
	vertical-align: baseline;
}

.reset .confirm {
	max-width: 0px;
	overflow: hidden;
	white-space: nowrap;
	transition: 500ms;
}

.reset.confirming .confirm {
	max-width: 500px;
}

.v-enter-active,
.v-leave-active {
	transition: transform 0.5s ease, opacity .3s ease;
}

.v-enter-from,
.v-leave-to {
	opacity: 0;
	transform: scale(.25);
}
</style>
