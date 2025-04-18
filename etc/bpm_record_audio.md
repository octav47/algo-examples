# Запись звука

```typescript
export async function recordMicrophoneWithWorklet(
  durationSeconds: number = 10
): Promise<AudioBuffer> {
  const stream = await navigator.mediaDevices.getUserMedia({
    audio: {
      channelCount: 1,
      sampleRate: 48000,
      sampleSize: 16,
      echoCancellation: false,
      noiseSuppression: false,
      autoGainControl: false,
    },
  });
  const audioContext = new AudioContext();
  const sampleRate = audioContext.sampleRate;

  const workletCode = `
    class RecorderProcessor extends AudioWorkletProcessor {
      constructor() {
        super();
      }
      process(inputs) {
        const input = inputs[0];
        if (input && input[0]) {
          const channelData = input[0];
          const copied = new Float32Array(channelData.length);
          copied.set(channelData);
          this.port.postMessage(copied);
        }
        return true;
      }
    }
    registerProcessor('recorder-processor', RecorderProcessor);
  `;

  const blob = new Blob([workletCode], { type: 'application/javascript' });
  const moduleURL = URL.createObjectURL(blob);
  await audioContext.audioWorklet.addModule(moduleURL);

  const workletNode = new AudioWorkletNode(audioContext, 'recorder-processor');
  const source = audioContext.createMediaStreamSource(stream);

  const chunks: Float32Array[] = [];
  let recordingLength = 0;
  const expectedSamples = durationSeconds * sampleRate;

  return new Promise<AudioBuffer>((resolve) => {
    workletNode.port.onmessage = (event) => {
      const chunk = event.data as Float32Array;
      chunks.push(chunk);
      recordingLength += chunk.length;

      if (recordingLength >= expectedSamples) {
        source.disconnect();
        workletNode.disconnect();
        stream.getTracks().forEach((t) => t.stop());

        const buffer = new Float32Array(recordingLength);
        let offset = 0;
        for (const chunk of chunks) {
          buffer.set(chunk, offset);
          offset += chunk.length;
        }

        const audioBuffer = audioContext.createBuffer(
          1,
          buffer.length,
          sampleRate,
        );
        audioBuffer.copyToChannel(buffer, 0, 0);
        resolve(audioBuffer);
      }
    };

    source.connect(workletNode);
    workletNode.connect(audioContext.destination);
  });
}
```
