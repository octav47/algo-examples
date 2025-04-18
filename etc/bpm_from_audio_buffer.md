# Вычисление темпа $`b_i`$

```typescript
function getMonoSignal(audioBuffer: AudioBuffer): Float32Array {
  if (audioBuffer.numberOfChannels === 1) {
    return audioBuffer.getChannelData(0);
  }

  const left = audioBuffer.getChannelData(0);
  const right = audioBuffer.getChannelData(1);
  const mono = new Float32Array(audioBuffer.length);
  for (let i = 0; i < audioBuffer.length; i++) {
    mono[i] = (left[i] + right[i]) / 2;
  }
  return mono;
}

function computeEnergy(signal: Float32Array, hopSize: number = 512): number[] {
  const energy: number[] = [];
  for (let i = 0; i < signal.length; i += hopSize) {
    let sum = 0;
    for (let j = i; j < i + hopSize && j < signal.length; j++) {
      sum += signal[j] * signal[j];
    }
    energy.push(sum);
  }
  return energy;
}

function detectOnsetTimes(
  energy: number[],
  hopSize: number,
  sampleRate: number,
  threshold: number = 1.5,
): number[] {
  const times: number[] = [];

  const mean = energy.reduce((a, b) => a + b) / energy.length;
  const thres = mean * threshold;

  for (let i = 1; i < energy.length - 1; i++) {
    const isPeak =
      energy[i] > thres &&
      energy[i] > energy[i - 1] &&
      energy[i] > energy[i + 1];
    if (isPeak) {
      const time = (i * hopSize) / sampleRate;
      times.push(time);
    }
  }

  return times;
}

function estimateBPMFromTimes(onsetTimes: number[]): number {
  if (onsetTimes.length < 2) return 0;

  const intervals: number[] = [];
  for (let i = 1; i < onsetTimes.length; i++) {
    const interval = onsetTimes[i] - onsetTimes[i - 1];
    if (interval > 0.25 && interval < 2.0) {
      intervals.push(interval);
    }
  }

  if (intervals.length === 0) return 0;

  const bpms = intervals.map((int) => 60 / int);
  const histogram: Record<number, number> = {};

  for (const bpm of bpms) {
    const rounded = Math.round(bpm);
    histogram[rounded] = (histogram[rounded] || 0) + 1;
  }

  const sorted = Object.entries(histogram).sort((a, b) => b[1] - a[1]);
  const [bpm] = sorted[0];

  return parseInt(bpm);
}

export function getBPMFromAudioBuffer(buffer: AudioBuffer): number {
  const signal = getMonoSignal(buffer);
  const hopSize = 512;
  const energy = computeEnergy(signal, hopSize);
  const onsets = detectOnsetTimes(energy, hopSize, buffer.sampleRate);
  const bpm = estimateBPMFromTimes(onsets);

  return bpm;
}

```
