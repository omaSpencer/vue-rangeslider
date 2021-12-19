<script setup>
import { onMounted, ref, reactive, computed } from 'vue';

const props = defineProps({
	min: {
		type: Number,
		required: false,
		default: 0,
	},
	max: {
		type: Number,
		required: false,
		default: 1000,
	},
	minValue: {
		type: Number,
		required: false,
		default: 0,
	},
	maxValue: {
		type: Number,
		required: false,
		default: 1000,
	},
	step: {
		type: Number,
		required: false,
		default: 1,
	},
	totalSteps: {
		type: Number,
		required: false,
		default: 100,
	},
	percentPerStep: {
		type: Number,
		required: false,
		default: 1,
	},
	double: {
		type: Boolean,
		required: false,
		default: false,
	},
});

const _vpcTrack = ref();
const trackHighlight = ref();
const track1 = ref();
const track2 = ref();

const min = ref(props.min);
const max = ref(props.max);
const minValue = ref(props.minValue);
const maxValue = ref(props.maxValue);
const step = ref(props.step);
const totalSteps = ref(props.totalSteps);
const percentPerStep = ref(props.percentPerStep);
const double = ref(props.double);

const trackWidth = ref(null);
const isDragging = ref(false);

const pos = reactive({
	curTrack: null,
});

const percentMinValue = computed(() => {
	return Math.round((minValue.value / max.value) * 100);
});
const percentMaxValue = computed(() => {
	return Math.round((maxValue.value / max.value) * 100);
});

const moveTrack = (track, ev) => {
	let percentInPx = getPercentInPx();

	let trackX = Math.round(_vpcTrack.value.getBoundingClientRect().left);
	let clientX = ev.clientX;
	let moveDiff = clientX - trackX;

	let moveInPct = moveDiff / percentInPx;
	// console.log(moveInPct)

	if (moveInPct < 1 || moveInPct > 100) return;
	let value =
		Math.round(moveInPct / percentPerStep.value) * step.value + min.value;

	if (double.value) {
		if (track === 'track1') {
			if (value >= maxValue.value - step.value) return;
			minValue.value = value;
			track1.value.style.left = moveInPct + '%';
		}
	}

	if (track === 'track2') {
		if (value <= minValue.value + step.value) return;
		maxValue.value = value;
		track2.value.style.left = moveInPct + '%';
	}

	setTrackHightlight();
};
const mousedown = (ev, track) => {
	if (isDragging.value) return;
	isDragging.value = true;
	pos.curTrack = track;
};

const touchstart = (ev, track) => {
	mousedown(ev, track);
};

const mouseup = (ev, track) => {
	if (!isDragging.value) return;
	isDragging.value = false;
};

const touchend = (ev, track) => {
	mouseup(ev, track);
};

const mousemove = (ev, track) => {
	if (!isDragging.value) return;
	moveTrack(track, ev);
};

const touchmove = (ev, track) => {
	mousemove(ev.changedTouches[0], track);
};

const valueToPercent = (value) => {
	return ((value - min.value) / step.value) * percentPerStep.value;
};

const setTrackHightlight = () => {
	trackHighlight.value.style.left = valueToPercent(minValue.value) + '%';
	trackHighlight.value.style.width =
		valueToPercent(maxValue.value) - valueToPercent(minValue.value) + '%';
};

const getPercentInPx = () => {
	let trackWidth = _vpcTrack.value.offsetWidth || 0;
	let oneStepInPx = trackWidth / totalSteps.value;
	// 1 percent in px
	let percentInPx = oneStepInPx / percentPerStep.value;

	return percentInPx;
};

const setClickMove = (ev) => {
	let track1Left = track1.value.getBoundingClientRect().left;
	let track2Left = track2.value.getBoundingClientRect().left;
	// console.log('track1Left', track1Left)
	if (ev.clientX < track1Left) {
		moveTrack('track1', ev);
	} else if (
		double.value &&
		ev.clientX - track1Left < track2Left - ev.clientX
	) {
		moveTrack('track1', ev);
	} else {
		moveTrack('track2', ev);
	}
};

onMounted(() => {
	// calc per step value
	totalSteps.value = (max.value - min.value) / step.value;

	// percent the track button to be moved on each step
	percentPerStep.value = 100 / totalSteps.value;
	// console.log('percentPerStep', this.percentPerStep)

	// set track1 initilal

	document.querySelector('.track1').style.left =
		valueToPercent(minValue.value) + '%';

	// track2 initial position
	document.querySelector('.track2').style.left =
		valueToPercent(minValue.value) + '%';

	// set initila track highlight
	setTrackHightlight();

	['mouseup', 'mousemove'].forEach((type) => {
		document.body.addEventListener(type, (ev) => {
			// ev.preventDefault();
			if (isDragging.value && pos.curTrack) {
				switch (type) {
					case 'mousedown':
						mousedown(ev, pos.curTrack);
						break;
					case 'mouseup':
						mouseup(ev, pos.curTrack);
						break;
					case 'mousemove':
						mousemove(ev, pos.curTrack);
						break;
					case 'touchstart':
						touchstart(ev, pos.curTrack);
						break;
					case 'touchmove':
						touchmove(ev, pos.curTrack);
						break;
					case 'touchend':
						touchend(ev, pos.curTrack);
						break;

					default:
						break;
				}
			}
		});
	});

	[
		'mousedown',
		'mouseup',
		'mousemove',
		'touchstart',
		'touchmove',
		'touchend',
	].forEach((type) => {
		document.querySelector('.track1').addEventListener(type, (ev) => {
			ev.stopPropagation();

			switch (type) {
				case 'mousedown':
					mousedown(ev, 'track1');
					break;
				case 'mouseup':
					mouseup(ev, 'track1');
					break;
				case 'mousemove':
					mousemove(ev, 'track1');
					break;
				case 'touchstart':
					touchstart(ev, 'track1');
					break;
				case 'touchmove':
					touchmove(ev, 'track1');
					break;
				case 'touchend':
					touchend(ev, 'track1');
					break;

				default:
					break;
			}
		});

		document.querySelector('.track2').addEventListener(type, (ev) => {
			ev.stopPropagation();
			switch (type) {
				case 'mousedown':
					mousedown(ev, 'track2');
					break;
				case 'mouseup':
					mouseup(ev, 'track2');
					break;
				case 'mousemove':
					mousemove(ev, 'track2');
					break;
				case 'touchstart':
					touchstart(ev, 'track2');
					break;
				case 'touchmove':
					touchmove(ev, 'track2');
					break;
				case 'touchend':
					touchend(ev, 'track2');
					break;

				default:
					break;
			}
		});
	});

	// on track click
	// determine direction based on click proximity
	// determine percent to move based on track.clientX - click.clientX
	document.querySelector('.track').addEventListener('click', function (ev) {
		ev.stopPropagation();
		setClickMove(ev);
	});

	document
		.querySelector('.track-highlight')
		.addEventListener('click', function (ev) {
			ev.stopPropagation();
			setClickMove(ev);
		});
});
</script>

<template>
	<div class="track-container">
		<span class="range-value min">{{ minValue }} </span>
		<span class="range-value max">{{ maxValue }}</span>
		<div class="track" ref="_vpcTrack"></div>
		<div class="track-highlight" ref="trackHighlight"></div>
		<button
			:class="`track-btn track1 ${!double ? 'opacity-0' : ''}`"
			ref="track1"
		>
			<span>{{ percentMinValue }}</span>
		</button>
		<button class="track-btn track2" ref="track2">
			<span>{{ percentMaxValue }}</span>
		</button>
	</div>
</template>

<style scoped>
.range-value {
	position: absolute;
	top: -2rem;
}
.range-value.min {
	left: 0;
}

.range-value.max {
	right: 0;
}
.track-container {
	width: 100%;
	position: relative;
	cursor: pointer;
	height: 0.5rem;
}

.track,
.track-highlight {
	display: block;
	position: absolute;
	height: 0.5rem;
	border-radius: 20px;
}

.track {
	background-color: #d2b7fa;
	width: 100%;
}

.track-highlight {
	background-color: #6200ee;
	z-index: 2;
	transform: rotate(180deg);
	width: 100%;
}

.track-btn {
	position: relative;
	-webkit-appearance: none;
	-moz-appearance: none;
	appearance: none;
	outline: none;
	cursor: pointer;
	display: block;
	position: absolute;
	z-index: 2;
	width: 1.5rem;
	height: 1.5rem;
	top: calc(-50% - 0.25rem);
	margin-left: -1rem;
	border: none;
	background-color: #6200ee;
	-ms-touch-action: pan-x;
	touch-action: pan-x;
	transition: box-shadow 0.3s ease-out, background-color 0.3s ease,
		-webkit-transform 0.3s ease-out;
	transition: transform 0.3s ease-out, box-shadow 0.3s ease-out,
		background-color 0.3s ease;
	transition: transform 0.3s ease-out, box-shadow 0.3s ease-out,
		background-color 0.3s ease, -webkit-transform 0.3s ease-out;
	border-radius: 50%;
}
.track-btn:focus {
	box-shadow: 0px 0px 0px 10px rgba(210, 183, 250, 0.5);
}
.track-btn span {
	position: absolute;
	bottom: calc(100% + 12px);
	left: 50%;
	transform: translateX(-50%);
}
.track-btn.opacity-0 {
	opacity: 0;
}
</style>
