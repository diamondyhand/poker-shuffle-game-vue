<script setup>
import { reactive, ref } from "vue";

const cards = reactive([
	{
		id: 1,
		label: "spades",
		open: false,
	},
	{
		id: 2,
		label: "hearts",
		open: false,
	},
	{
		id: 3,
		label: "clubs",
		open: false,
	},
	{
		id: 4,
		label: "diamonds",
		open: false,
	},
]);
const symbols = reactive([
	{ label: "spades", symbol: "♠" },
	{ label: "hearts", symbol: "♥" },
	{ label: "diamonds", symbol: "♦" },
	{ label: "clubs", symbol: "♣" },
]);
const state = ref("Press the card to start a new game!");
const gather = ref(true),
	question = ref(null),
	mode = ref(""),
	count = ref(0);

function getSymbol(label) {
	let result = symbols.find((symbol) => symbol.label === label);
	return result ? result.symbol : label;
}
function shuffle() {
	let newOrder = [1, 2, 3, 4].sort(() => Math.random() - 0.5);
	cards.forEach((card, index) => (card.id = newOrder[index]));
}
function turnAllCard(state) {
	cards.forEach((card) => (card.open = state));
}
let startShuffle = () => {
	shuffle();
	if (count.value++ < 6) {
		setTimeout(startShuffle, 300);
	} else {
		state.value = `Please pick out ${question.value.label} ${question.value.symbol}!!`;
		mode.value = "answer";
	}
};
function startGame() {
	mode.value = "stratgame";
	question.value = symbols[parseInt(Math.random() * 3)];
	turnAllCard(false);
	gather.value = true;
	state.value = "Ready...";
	setTimeout(() => {
		gather.value = false;
		state.value = "Your mission is...";
	}, 1000);
	setTimeout(() => {
		turnAllCard(true);
		state.value = `Find ${question.value.label} ${question.value.symbol}!!`;
	}, 2000);
	setTimeout(() => {
		turnAllCard(false);
		state.value = `Get Ready...`;
	}, 4000);

	count.value = 0;
	setTimeout(() => {
		startShuffle();
	}, 6000);
}
function getCard(label) {
	return cards.find((card) => card.label === label);
}
function openCard(card) {
	if (mode.value === "") startGame();
	if (mode.value !== "answer") return;
	card.open = !card.open;
	if (card.label === question.value.label) {
		state.value = `🤙🤙🤙🦀  You get the ${question.value.label} ${question.value.symbol}!!!  🦀🤟🤟🤟`;
	} else {
		state.value = "👎👎👎🐧 You lose!!! 🐧 👎👎👎";
		let card = getCard(question.value.label);
		setTimeout(() => (card.open = true), 1000);
	}
	setTimeout(startGame, 3000);
}
</script>

<template>
	<div class="control">
		<div>
			<input type="checkbox" id="gatherControl" v-model="gather" />
			<label for="gatherControl">Gather</label>
		</div>
		<!-- <input type="checkbox" v-for="card in cards" v-model="card.open" /> -->

		<div class="btnContainer">
			<button @click="shuffle">Shuffle</button>
			<button @click="startGame">Start Game</button>
		</div>
	</div>

	<p class="state">{{ state }}</p>

	<ul class="cards" :class="{ gather: gather }">
		<li
			class="card"
			v-for="card in cards"
			:class="{ open: card.open }"
			:data-order="card.id"
			:key="card.label"
			@click="openCard(card)"
		>
			<div class="face back"></div>
			<div class="face front">{{ getSymbol(card.label) }}</div>
			<p style="display: absolute; z-index: 999">
				{{ card.id }}
			</p>
		</li>
	</ul>
</template>

<style lang="scss" scoped>
@mixin size($width, $height: $width) {
	width: $width;
	height: $height;
}
$cardWidth: 15vw;
@mixin setLeftPosition($left, $width: $cardWidth) {
	left: calc(#{$left} - #{$width} / 2);
}
p.state {
	text-align: center;
	font-size: clamp(1.2rem, 2.4vw, 2.4rem);
}
.control {
	margin-top: 2rem;
	display: flex;
	justify-content: center;
	align-items: center;
	gap: 2.4rem;
	.btnContainer {
		display: flex;
		justify-content: center;
		align-items: center;
		gap: 1.5rem;
		button {
			display: block;
			padding: 0.24rem 0.8rem;
			border: none;
			border-radius: 8px;
			cursor: pointer;
			font-size: 1.2rem;
			background-color: #9ace98;
			&:active {
				transform: translateY(0.24rem);
			}
		}
	}
}
.cards {
	@include size(100vw, 40vh);
	list-style: none;
	transition: 0.8s;
	perspective: 800px;
	margin: 0 auto;
	.card {
		@include size($cardWidth, $cardWidth * 1.4);
		background-color: #fff;
		position: absolute;
		top: 35%;
		@include setLeftPosition(50%);
		border-radius: 5px;
		box-shadow: 0 0.8rem 2rem rgb(0 0 0 / 0.24);
		transform-style: preserve-3d;
		transition: left 0.5s, top 0.5s, transform 0.8s;
		cursor: pointer;

		.face {
			background-color: #fff;
			color: #111;
			position: absolute;
			inset: 0;
			border-radius: 5px;
			backface-visibility: hidden;

			&.front {
				transform: rotateY(180deg);
				display: flex;
				justify-content: center;
				align-items: center;
				font-size: 60px;
			}
			&.back {
				$colorRed: #ed4747;
				&::before {
					content: "";
					display: block;
					position: absolute;
					inset: 10px;
					border: 3px solid $colorRed;
					background-image: linear-gradient(
							-60deg,
							transparent 40%,
							$colorRed 40%,
							$colorRed 60%,
							transparent 60%
						),
						linear-gradient(
							60deg,
							transparent 40%,
							$colorRed 40%,
							$colorRed 60%,
							transparent 60%
						);
					background-size: 8px 8px;
				}
			}
		}

		&.open {
			transform: rotateY(180deg);
		}
		&:hover {
			top: 30%;
		}
		@for $i from 1 through 4 {
			&[data-order="#{$i}"] {
				@include setLeftPosition(#{$i * 20%});
			}
		}
	}

	&.gather {
		cursor: pointer;
		width: calc(#{$cardWidth} + 8rem);
		.card {
			@include setLeftPosition(50%);
			top: 20%;
			transform: rotate(360deg);
		}
		&:hover {
			@for $i from 1 through 4 {
				.card[data-order="#{$i}"] {
					transform: rotate(#{$i * 5 + 350}deg);
				}
			}
		}
	}
}
</style>
