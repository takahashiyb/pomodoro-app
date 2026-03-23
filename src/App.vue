<script setup lang="ts">
import { ref, watch } from 'vue'

interface Timer {
  name: string
  duration: number
  remaining: number
  minutes(): string
  seconds(): string
  percent(): number
}

const timers: Timer[] = [
  {
    name: 'pomodoro',
    duration: 1500,
    remaining: 1500,
    minutes() {
      return Math.floor(this.remaining / 60)
        .toString()
        .padStart(2, '0')
    },
    seconds() {
      return (this.remaining % 60).toString().padStart(2, '0')
    },
    percent() {
      return (this.remaining / this.duration) * 100
    },
  },
  {
    name: 'short break',
    duration: 300,
    remaining: 300,
    minutes() {
      return Math.floor(this.remaining / 60)
        .toString()
        .padStart(2, '0')
    },
    seconds() {
      return (this.remaining % 60).toString().padStart(2, '0')
    },
    percent() {
      return (this.remaining / this.duration) * 100
    },
  },
  {
    name: 'long break',
    duration: 900,
    remaining: 900,
    minutes() {
      return Math.floor(this.remaining / 60)
        .toString()
        .padStart(2, '0')
    },
    seconds() {
      return (this.remaining % 60).toString().padStart(2, '0')
    },
    percent() {
      return (this.remaining / this.duration) * 100
    },
  },
]

const current = ref<Timer>(timers[0] as Timer)

function setTimer(timer: Timer) {
  resetDuration()

  current.value = timer
}

function convertToSeconds(minutes: number) {
  return minutes * 60
}

const openDialog = ref<boolean>(false)

const font = ref<string>('font-3')

const color = ref<string>('red')

const isTicking = ref<boolean>(false)

let startTime: number | null

let pauseTime: number | null

let interval: ReturnType<typeof setInterval>

function resetDuration() {
  clearInterval(interval)

  isTicking.value = false

  startTime = null

  pauseTime = null

  current.value.remaining = current.value.duration
}

function runDuration() {
  isTicking.value = true

  if (!startTime) {
    startTime = Date.now()
  }

  interval = setInterval(() => {
    current.value.remaining--
  }, 1000)

  watch(
    () => current.value.remaining,
    (newVal) => {
      if (newVal === 0) {
        resetDuration()
      }
    },
  )
}

function pauseDuration() {
  console.log(isTicking)
  console.log(startTime)
  if (!isTicking || !startTime) return

  isTicking.value = false

  pauseTime = Date.now()

  clearInterval(interval)
}
</script>

<template>
  <dialog :open="openDialog"></dialog>
  <header>
    <img src="./assets/icons/logo.svg" alt="" />
  </header>
  <main class="purple" :class="[font, color]">
    <ul class="container__type">
      <li
        class="type"
        :class="{ selected: current.name === timer.name }"
        v-for="timer in timers"
        @click="setTimer(timer)"
      >
        {{ timer.name }}
      </li>
    </ul>
    <section class="container__timer">
      <div class="timer">
        <svg>
          <circle
            r="calc(50% - 8px)"
            cx="50%"
            cy="50%"
            pathLength="100"
            stroke-dasharray="100"
            :stroke-dashoffset="`calc(100
            -
        ${current.percent()})`"
            ;
          />
        </svg>
        <span class="text__duration">{{ current.minutes() }}:{{ current.seconds() }}</span>
        <span class="text__command" @click="pauseDuration" v-if="isTicking">PAUSE</span>
        <span class="text__command" @click="runDuration" v-if="!isTicking">RESTART</span>
      </div>
    </section>
    <section class="container__settings" @click="openDialog = true">
      <img src="./assets/icons/icon-settings.svg" alt="" />
    </section>
  </main>
</template>
<style lang="scss">
@use '@/assets/styles/main.scss' as v;

html {
  height: 100%;
  background-color: v.$blue-850;
  display: grid;
  place-items: center;

  padding: v.$spacing-0200;
}
</style>
<style scoped lang="scss">
@use '@/assets/styles/main.scss' as v;
@use '@/assets/styles/functions.scss' as f;

header,
.container__settings {
  display: grid;
  justify-content: center;

  font-family: 'Roboto Sans';
}

main {
  display: flex;
  flex-direction: column;
  gap: v.$spacing-0600;
}

.container__type {
  min-width: 100%;
  background-color: rgba(v.$blue-900, 100%);
  list-style: none;

  display: grid;
  grid-template-columns: repeat(3, 1fr);
  align-items: center;

  padding: v.$spacing-0100;

  border-radius: 9em;
}

.type {
  text-align: center;

  padding: v.$spacing-0200;

  border-radius: 9em;
}

.type {
  color: rgba(v.$blue-100, 100%);

  cursor: pointer;
}

.type.selected {
  color: rgba(v.$blue-850, 100%);
}

main.font-1 .type {
  font-family: v.$font-1;

  @include f.responsive-type(v.$font-1-text-3m, v.$font-1-text-3, v.$font-1-text-3);
}
main.font-2 .type {
  font-family: v.$font-2;

  @include f.responsive-type(v.$font-2-text-3m, v.$font-2-text-3, v.$font-2-text-3);
}
main.font-3 .type {
  font-family: v.$font-3;

  @include f.responsive-type(v.$font-3-text-3m, v.$font-3-text-3, v.$font-3-text-3);
}

main.red .type.selected {
  background-color: v.$red-400;
}

main.cyan .type.selected {
  background-color: v.$cyan-300;
}
main.purple .type.selected {
  background-color: v.$purple-400;
}

.container__timer {
  aspect-ratio: 1;
  background-image: linear-gradient(to top left, v.$gradient-l, v.$gradient-d);
  border-radius: 50%;

  padding: v.$spacing-0200;

  box-shadow:
    50px 50px 100px v.$gradient-d,
    -50px -50px 100px v.$gradient-l;
}

.timer {
  background-color: v.$blue-900;
  aspect-ratio: 1;
  min-width: 100%;

  padding: v.$spacing-0100;

  border-radius: 50%;

  display: grid;
  grid-template-rows: repeat(3, 1fr);
  gap: v.$spacing-0100;
}

svg {
  height: 100%;
  width: 100%;

  stroke-width: 8px;
  fill: none;
  stroke-linecap: round;

  transform: rotateZ(-90deg) rotateX(180deg);

  grid-column: 1/-1;
  grid-row: 1/-1;

  z-index: 0;
}

main.red svg {
  stroke: v.$red-400;
}

main.cyan svg {
  stroke: v.$cyan-300;
}
main.purple svg {
  stroke: v.$purple-400;
}

.text__duration {
  color: white;
  align-self: center;
  justify-self: center;

  padding: 8px;

  grid-column: 1/-1;
  grid-row: 2;

  z-index: 1;
}

main.font-1 .text__duration {
  font-family: v.$font-1;

  @include f.responsive-type(v.$font-1-text-1m, v.$font-1-text-1, v.$font-1-text-1);
}
main.font-2 .text__duration {
  font-family: v.$font-2;

  @include f.responsive-type(v.$font-2-text-1m, v.$font-2-text-1, v.$font-2-text-1);
}
main.font-3 .text__duration {
  font-family: v.$font-3;

  @include f.responsive-type(v.$font-3-text-1m, v.$font-3-text-1, v.$font-3-text-1);
}

.text__command {
  color: white;
  align-self: flex-start;
  justify-self: center;
  cursor: pointer;

  grid-column: 1/-1;
  grid-row: 3;

  z-index: 1;
}

main.font-1 .text__command {
  font-family: v.$font-1;

  @include f.responsive-type(v.$font-1-text-2m, v.$font-1-text-2, v.$font-1-text-2);
}
main.font-2 .text__command {
  font-family: v.$font-2;

  @include f.responsive-type(v.$font-2-text-2m, v.$font-2-text-2, v.$font-2-text-2);
}
main.font-3 .text__command {
  font-family: v.$font-3;

  @include f.responsive-type(v.$font-3-text-2m, v.$font-3-text-2, v.$font-3-text-2);
}
</style>
