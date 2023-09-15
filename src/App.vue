<script>
// import HelloWorld from './components/HelloWorld.vue'
import DurationInput from './components/DurationInput.vue'
import soundsData from './sounds'
const sounds = Object.fromEntries(soundsData.map(sound => [sound.name, { ...sound, audio: new Audio('/alarms/'+sound.filename)}]))

export default {
	name: 'App',
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
			view: 'menu',
			soundName: sounds['Classic'].name,
			samplingAudio: null,
			showSettings: false
		}
	},
	components: {
		// HelloWorld,
		// TheWelcome,
		DurationInput
	},
	beforeMount() {
		this.animate();
	},
	beforeUnmount() {
		this.reset();
	},

	methods: {
		start() {
			this.showSettings = false;
			this.view = 'timer';
			this.addTime();
		},
		onClickReset() {
			if (!this.timeRemaining || this.confirmingReset) {
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
			this.view = 'menu';
			this.timeEnd = null;
			this.timeRemaining = 0;
			this.totalTime = 0;
			clearInterval(this.timeout);
			cancelAnimationFrame(this.animate);
			if (this.alarmAudio) {
				this.alarmAudio.pause();
			}
		},
		addTime() {
			clearTimeout(this.timeout);
			if (this.alarmAudio) {
				this.alarmAudio.pause();
			}

			const maxEnd = this.maxTime ? this.maxTime + Date.now() : undefined
			if (this.tapMode === 'additive') {
				this.timeEnd = Math.min(maxEnd || Infinity, (this.timeEnd || Date.now()) + this.tapValue);
			}
			else if (this.tapMode === 'maximum') {
				this.timeEnd = maxEnd;
			}
			this.totalTime = this.timeEnd - Date.now();
			this.timeout = setTimeout(this.onTimeEnd, this.timeEnd - Date.now());

			this.addingTime = true;
			setTimeout(() => {
				this.addingTime = false;
			}, 150);
		},
		onTimeEnd() {
			this.timeEnd = null;
			this.playAlarm();
		},
		playAlarm() {
			if (this.alarmAudio) {
				this.alarmAudio.currentTime = 0;
				this.alarmAudio.loop = true;
				this.alarmAudio.play();
			}
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
		changeSound(event) {
			const newSoundName = event.target.value;
			this.stopSample();
			this.soundName = newSoundName;
		},
		async sampleSound() {
			if (this.samplingAudio) {
				this.stopSample();
			}
			this.samplingAudio = sounds[this.soundName].audio;
			await this.samplingAudio.play();
			this.samplingAudio.addEventListener('ended', this.stopSample)
		},
		async stopSample() {
			if(this.samplingAudio) {
				this.samplingAudio.pause();
				this.samplingAudio.currentTime = 0;
				this.samplingAudio = null;
			}
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
				return 'var(--color-text)';
			}
			return this.timeRemaining <= Math.min(11000, this.totalTime / 2) ? 'var(--red)' : 'var(--green)'
		},
		sounds: () => sounds,
		alarmAudio() {
			return sounds[this.soundName]?.audio
		},
	},
}

</script>`

<template>
	<div class="app-container">
		<div
			class="menu"
			v-if="view === 'menu'"
		>
			<div :style="{
				height: '50%',
				display: 'flex',
				alignItems: 'center',
			}">
				<h1 class="title">
					TAPPY TIMER
				</h1>
			</div>
			<div class="start-button"
				v-if="view === 'menu'"
				@click="start"
				:style="{
					filter: `drop-shadow(0 0 50px var(--green))`
				}">
				Start
			</div>

			<div :style="{
				height: '50%',
				display: 'flex',
				alignItems: 'center',
			}">
				<div class="settings"
					:class="{ open: showSettings}"
					v-if="view === 'menu'"
				>
					<div class="settings-title"
						@click="showSettings = !showSettings"
					>
						Settings
						<span class="fa" :class="showSettings ? 'fa-chevron-down' : 'fa-chevron-right'"></span>
					</div>
					<div class="settings-wrapper">
						<div class="row">
							<label>Sound:</label>
							<span>
								<span v-if="samplingAudio" class="fa fa-times"  @click="stopSample"></span>
								<span v-else-if="soundName" class="fa fa-volume-up" @click="sampleSound"></span>
								<select
									:value="soundName"
									@change="changeSound"
								>
									<option key="none" :value="null" label="None" />
									<option v-for="sound in sounds" :key="sound.name" :value="sound.name">{{ sound.name }}</option>
								</select>
							</span>
						</div>
						<div class="row">
							<label>Mode:</label>
							<select
								v-model="tapMode"
							>
								<option key="additive" value="additive">Additive</option>
								<option key="maximum" value="maximum">Maximum</option>
							</select>
						</div>
						<div class="row" v-if="tapMode === 'additive'">
							<label>Increment:</label>
							<DurationInput
								:value="tapValue"
								@didChange="v => tapValue = v"
							/>
						</div>
						<div class="row">
							<label>Max Time:</label>
							<DurationInput
								:value="maxTime"
								@didChange="v => maxTime = v"
							/>
						</div>
					</div>
				</div>
			</div>
			
		</div>
		<div class="tap-message"
			v-if="view === 'timer'"
		>
			Tap anywhere to add time
		</div>
		<div class="reset"
			v-if="view === 'timer'"
			:class="{ confirming: confirmingReset }"
			@click.stop="onClickReset">
			<span class="icon fa fa-times"></span>
			<span class="confirm">&nbsp;&nbsp;Press again to close</span>
		</div>
		<Transition>
			<div class="svg-wrapper"
				:class="{ active: addingTime }"
				v-if="view === 'timer'"
				@click="addTime">
				<svg viewBox="0 0 200 200"
					preserve-aspect-ratio
					:style="{
						filter: `drop-shadow(0 0 50px ${timerColor})`
					}">
					<circle cx="100"
						cy="100"
						r="80"
						fill="var(--color-background)"
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
						fill="var(--color-heading)">
						{{ fmtMSS(timeRemaining) }}
					</text>
				</svg>
			</div>
		</Transition>
	</div>
</template>

<style scoped>
.app-container {
	position: relative;
	width: 100vw;
	height: calc(100vh - 50px);
	user-select: none;
}

.menu {
	display: flex;
	flex-direction: column;
	align-items: center;
	justify-content: space-between;
	height: 100%;
}

.title {
	font-size: 3em;
	font-weight: bold;
	color: var(--color-heading);
}

.start-button {
	text-align: center;
	background-color: var(--green);
	color: white;
	font-weight: bold;
	border-radius: 50%;
	font-size: 2.5em;
	min-height: 5em;
	min-width: 5em;
	max-height: 5em;
	max-width: 5em;
	z-index: 2;
	cursor: pointer;
	display: flex;
	place-content: center;
	place-items: center;
}

.settings {
	text-align: center;
	width: 100%;
	transition: 500ms;
}


.settings .settings-title {
	font-size: 1.5em;
	margin: .5em;
	cursor: pointer;
	display: flex;
	align-items: center;
	justify-content: center;
	gap: .5em;
}

.settings .settings-title .fa {
	font-size: .75em;
}

.settings-wrapper {
	display: flex;
	flex-direction: column;
	gap: .5em;
	max-height: 0;
	overflow: hidden;
	transition: max-height 500ms;
}

.settings.open .settings-wrapper {
	max-height: 100vh;
}
.settings-wrapper .row {
	display: flex;
	width: 100%;
	justify-content: space-between;
	align-items: center;
	gap: 4em;
}

.settings .row .fa {
	padding: .5em;
	cursor: pointer;
	margin-left: -1em;
}

.settings .row .fa:active {
	transform: scale(1.2);
}

.tap-message {
	text-align: center;
	position: relative;
	top: 2em;
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
