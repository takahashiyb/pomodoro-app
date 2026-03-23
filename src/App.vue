<script setup lang="ts">
import { ref, watch } from 'vue'

// # # # # # Timer Data # # # # # # //

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

function setDuration(minutes: number, timer: Timer) {
  const time = convertToSeconds(minutes)

  timer.duration = time
  timer.remaining = time
}

// # # # # # Styles # # # # # # //

const fontList = [
  { name: 'Kumbh Sans', id: 'font-1', short: 'k' },
  { name: 'Roboto Slab', id: 'font-2', short: 'r' },
  { name: 'Space Mono', id: 'font-3', short: 's' },
]

const colorList = [{ name: 'red' }, { name: 'cyan' }, { name: 'purple' }]

const setFont = ref<string>('font-3')

const setColor = ref<string>('red')

// # # # # # Timer # # # # # # //

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
      if (newVal <= 0) {
        resetDuration()
      }
    },
  )
}

function pauseDuration() {
  if (!isTicking || !startTime) return

  isTicking.value = false

  pauseTime = Date.now()

  clearInterval(interval)
}

// # # # # # Dialog / Settings # # # # # # //

const isDialogOpen = ref<boolean>(false)

function openDialog() {
  pauseDuration()

  isDialogOpen.value = true
}

function applyChanges(e: Event) {
  const form = e.target as HTMLFormElement
  const newTimers = [
    { pomodoro: (form.elements.namedItem('pomodoro') as HTMLInputElement).value },
    { 'short break': (form.elements.namedItem('short break') as HTMLInputElement).value },
    { 'long break': (form.elements.namedItem('long break') as HTMLInputElement).value },
  ]
  for (let i = 0; i < newTimers.length; i++) {
    const timer = timers.find((j) => j.name === Object.keys(newTimers[i]!)[0])
    setDuration(Object.values(newTimers[i]!)[0], timer!)
  }

  const font = (form.elements.namedItem('font') as HTMLInputElement).value
  setFont.value = font

  const color = (form.elements.namedItem('color') as HTMLInputElement).value
  setColor.value = color

  isDialogOpen.value = false
}
</script>

<template>
  <dialog :open="isDialogOpen">
    <form @submit.prevent="applyChanges">
      <header>
        <span class="text__settings--h">Settings</span>
        <img
          src="@/assets/icons/icon-close.svg"
          alt="close settings display"
          @click="isDialogOpen = false"
        />
      </header>
      <br />
      <section class="container__settings container__settings--time">
        <span class="text__settings--h2">TIME (MINUTES)</span>
        <div>
          <label class="text__settings--h3" v-for="timer in timers">
            <span>{{ timer.name }}</span>
            <input type="number" :value="Number(timer.minutes())" :name="timer.name" />
          </label>
        </div>
      </section>
      <br />
      <section class="container__settings container__settings--style">
        <span class="text__settings--h2">FONT</span>
        <div>
          <label
            :class="[
              'settings__option',
              `settings__option--font`,
              `settings__option--${font.short}`,
            ]"
            v-for="font in fontList"
          >
            <input type="radio" name="font" :checked="font.id === setFont" :value="font.id" />
            <span class="sr-only">click to select the font: {{ font.name }}</span>
            <span aria-hidden="true">Aa</span>
          </label>
        </div>
      </section>
      <br />
      <section class="container__settings container__settings--style">
        <span class="text__settings--h2">COLOR</span>
        <div>
          <label
            :class="[
              'settings__option',
              'settings__option--color',
              `settings__option--${color.name}`,
            ]"
            v-for="color in colorList"
          >
            <input
              type="radio"
              name="color"
              :checked="color.name === setColor"
              :value="color.name"
            />
            <span class="sr-only">click to select the color: {{ color.name }}</span>
            <img src="./assets/icons/icon-checkmark.svg" alt="checkmark icon" />
          </label>
        </div>
      </section>
      <div class="button__settings" type="submit" @submit.prevent="applyChanges">
        <button type="submit">Apply</button>
      </div>
    </form>
  </dialog>
  <header class="header__main">
    <img src="./assets/icons/logo.svg" alt="" />
  </header>
  <main :class="[setFont, setColor]">
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
    <section class="container__settings" @click="openDialog">
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

.header__main {
  padding-bottom: 20%;
}

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

  z-index: 1;
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

dialog {
  border: 0;
  border-radius: 15px;

  padding-bottom: 0;

  position: absolute;
  top: 50%;
  translate: 0 -50%;

  z-index: 99;
}

form header {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

form header img {
  cursor: pointer;
}

.text__settings--h {
  color: rgba(v.$blue-900, 100%);

  font-family: v.$font-1;

  @include f.flat-type(v.$settings-text-1);
}

.text__settings--h2 {
  color: rgba(v.$blue-900, 100%);

  font-family: v.$font-1;

  @include f.flat-type(v.$settings-text-3);
}

.container__settings--time {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: v.$spacing-0200;
}

.container__settings--style {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: v.$spacing-0200;
}

.container__settings--style div {
  height: 40px;
  width: min-content;

  display: flex;
  justify-content: space-between;
  gap: v.$spacing-0200;
}

.text__settings--h3 {
  font-family: v.$font-1;

  color: rgba(v.$blue-850, 100%);

  @include f.flat-type(v.$settings-text-4);
}

.container__settings--time div {
  display: flex;
  justify-content: space-between;
  flex-direction: column;
  align-items: center;
  gap: v.$spacing-0200;
}

.container__settings--time div label {
  display: flex;
  justify-content: space-between;
  align-items: center;
  gap: v.$spacing-0200;
}

.container__settings--time div label input {
  background-color: rgba(v.$blue-050, 100%);
  padding: 12px;
  border: none;
  border-radius: 10px;
  outline: none;
}

.settings__option {
  aspect-ratio: 1;
  background-color: v.$blue-050;
  color: v.$blue-850;
  cursor: pointer;

  border-radius: 50%;

  display: grid;
  place-items: center;
}

input[type='radio'] {
  display: none;
}

.settings__option--font:has(input:checked) {
  background-color: v.$blue-900;
  color: v.$white;
}

.settings__option--k {
  font-family: v.$font-1;

  @include f.flat-type(v.$settings-text-2-k);
}

.settings__option--r {
  font-family: v.$font-2;

  @include f.flat-type(v.$settings-text-2-r);
}

.settings__option--s {
  font-family: v.$font-3;

  @include f.flat-type(v.$settings-text-2-s);
}

.settings__option--color img {
  opacity: 0;
}

.settings__option--color:has(input:checked) img {
  opacity: 100;
}

.settings__option--red {
  background-color: rgba(v.$red-400, 100%);
}

.settings__option--cyan {
  background-color: rgba(v.$cyan-300, 100%);
}

.settings__option--purple {
  background-color: rgba(v.$purple-400, 100%);
}

.button__settings {
  display: flex;
  justify-content: center;

  translate: 0 50%;
}

.button__settings button {
  background-color: rgba(v.$red-400, 100%);
  width: min-content;
  color: rgba(v.$white, 100%);
  cursor: pointer;

  font-family: v.$font-1;

  @include f.flat-type(v.$settings-text-3);

  border-radius: 9em;
  border: none;
  outline: none;

  padding: v.$spacing-0200;
  padding-inline: v.$spacing-0600;
}
</style>
