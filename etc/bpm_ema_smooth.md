# Уточнение итогово значения темпа $`B`$

```typescript
let smoothedBPM = null;

export function ema(newBPM: number) {
  const alpha = 0.2;

  smoothedBPM =
    smoothedBPM === null
      ? newBPM
      : smoothedBPM * (1 - alpha) +
      newBPM * alpha;

  return smoothedBPM;
}

```
