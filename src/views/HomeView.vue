<script setup>
import SquareNoAnimation from '../components/SquareNoAnimation.vue'

const scaleFrequencies = [130.81, 146.83, 164.81, 196.0, 220.0]

const audioContext = new (window.AudioContext || window.webkitAudioContext)()
let activeOscillator = null
let activeGainNode = null

function playTone() {
  if (audioContext.state === 'suspended') {
    audioContext.resume().then(() => triggerSound())
  } else {
    triggerSound()
  }
}

function triggerSound() {
  if (activeOscillator) {
    const now = audioContext.currentTime
    activeGainNode.gain.cancelScheduledValues(now)
    activeGainNode.gain.setValueAtTime(activeGainNode.gain.value, now)
    activeGainNode.gain.linearRampToValueAtTime(0, now + 0.1)
    activeOscillator.stop(now + 0.1)
  }

  const oscillator = audioContext.createOscillator()
  const gainNode = audioContext.createGain()

  const randomFrequency = scaleFrequencies[Math.floor(Math.random() * scaleFrequencies.length)]

  oscillator.type = 'sine'
  oscillator.frequency.setValueAtTime(randomFrequency, audioContext.currentTime)

  oscillator.connect(gainNode)
  gainNode.connect(audioContext.destination)

  const now = audioContext.currentTime
  const attack = 0.05
  const decay = 0.1
  const sustain = 0.3
  const release = 0.6

  gainNode.gain.setValueAtTime(0, now)
  gainNode.gain.linearRampToValueAtTime(0.5, now + attack)
  gainNode.gain.linearRampToValueAtTime(sustain, now + attack + decay)
  gainNode.gain.linearRampToValueAtTime(0, now + attack + decay + release)

  oscillator.start(now)
  oscillator.stop(now + attack + decay + release)

  activeOscillator = oscillator
  activeGainNode = gainNode
}
</script>

<template>
  <main>
    <div class="grid-container">
      <SquareNoAnimation
        v-for="n in 60"
        :key="n"
        class="square"
        @mouseover="playTone"
        @touchstart="playTone"
        @click="playTone"
      />
    </div>
  </main>
</template>

<style scoped>
main {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100vh;
  width: 100vw;
  background-color: rgba(238, 223, 223, 0.83);
  background-color: rgba(255, 255, 255, 0.83);
}

.grid-container {
  display: grid;
  grid-template-columns: repeat(10, 1fr);
  grid-auto-rows: auto;
  gap: 20px;
  width: 80vw;
  height: auto;
}

.square {
  position: relative;
  aspect-ratio: 1 / 1;
  transform-origin: center;
  box-shadow:
    0 4px 6px rgba(142, 93, 93, 0.862),
    0 6px 12px rgba(7, 2, 2, 0.612),
    0 12px 16px rgba(0, 0, 0, 0.378);
  background-color: rgba(255, 255, 255, 0.83);

  opacity: 0.85;
  transition:
    transform 0.3s ease,
    background-color 0.3s ease;
}

.square:hover {
  transform: scale(1.05) rotate(12deg);
  background-color: rgba(255, 182, 193, 0.153);
  cursor: pointer;
}
@media (max-width: 1024px) {
  .grid-container {
    grid-template-columns: repeat(8, 1fr);
    gap: 15px;
    width: 85vw;
  }
}

@media (max-width: 768px) {
  .grid-container {
    grid-template-columns: repeat(6, 1fr);
    gap: 12px;
    width: 90vw;
  }
}

@media (max-width: 480px) {
  .grid-container {
    grid-template-columns: repeat(4, 1fr);
    gap: 10px;
    width: 95vw;
  }
}
</style>
