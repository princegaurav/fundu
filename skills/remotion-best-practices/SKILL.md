---
name: remotion-best-practices
description: Best practices for Remotion - Video creation in React
metadata:
  tags: remotion, video, react, animation, composition
---

## When to use

Use this skill whenever you are dealing with Remotion code to obtain the domain-specific knowledge.

## Captions

When dealing with captions or subtitles, refer to the subtitles rules for more information.

## Using FFmpeg

For some video operations, such as trimming videos or detecting silence, FFmpeg should be used.

## Audio visualization

When needing to visualize audio (spectrum bars, waveforms, bass-reactive effects), use Remotion's audio visualization APIs.

## Sound effects

When needing to use sound effects, use Remotion's audio APIs.

## Core Concepts

### Compositions

A Remotion composition defines a video:

```tsx
import { Composition } from 'remotion';

export const RemotionRoot = () => {
  return (
    <Composition
      id="MyVideo"
      component={MyVideo}
      durationInFrames={120}
      fps={30}
      width={1920}
      height={1080}
    />
  );
};
```

### Animations

Use `useCurrentFrame()` and `interpolate()` for animations:

```tsx
import { useCurrentFrame, interpolate } from 'remotion';

const MyComponent = () => {
  const frame = useCurrentFrame();
  const opacity = interpolate(frame, [0, 30], [0, 1], {
    extrapolateRight: 'clamp',
  });
  return <div style={{ opacity }}>Hello</div>;
};
```

### Spring animations

```tsx
import { spring, useCurrentFrame, useVideoConfig } from 'remotion';

const frame = useCurrentFrame();
const { fps } = useVideoConfig();

const scale = spring({ frame, fps, config: { damping: 10 } });
```

### Sequencing

Use `<Sequence>` to offset and trim animations:

```tsx
import { Sequence } from 'remotion';

<Sequence from={30} durationInFrames={60}>
  <MyComponent />
</Sequence>
```

### Series

Use `<Series>` for sequential animations:

```tsx
import { Series } from 'remotion';

<Series>
  <Series.Sequence durationInFrames={30}><First /></Series.Sequence>
  <Series.Sequence durationInFrames={30}><Second /></Series.Sequence>
</Series>
```

## Key Rules

### Rule Categories

| Category | Key Points |
|----------|-----------|
| **Compositions** | Define video dimensions, fps, duration |
| **Animations** | Use `interpolate()` with `extrapolateRight: 'clamp'` |
| **Timing** | `spring()` for natural motion, `interpolate()` for linear |
| **Sequencing** | `<Sequence>` for timing, `<Series>` for chaining |
| **Assets** | Use `staticFile()` for local assets |
| **Audio** | `<Audio>` component, `useCurrentFrame()` for sync |
| **Fonts** | Load fonts with `loadFont()` from `@remotion/google-fonts` |
| **3D** | Use `@remotion/three` for Three.js integration |
| **Transitions** | Use `@remotion/transitions` for scene transitions |

### Assets

```tsx
import { Img, staticFile } from 'remotion';

<Img src={staticFile('logo.png')} />
```

### Audio

```tsx
import { Audio, staticFile } from 'remotion';

<Audio src={staticFile('music.mp3')} volume={0.5} />
```

### Video

```tsx
import { Video, staticFile } from 'remotion';

<Video src={staticFile('clip.mp4')} startFrom={30} endAt={90} />
```

## Rendering

```bash
# Render to MP4
npx remotion render MyVideo out/video.mp4

# Render a still
npx remotion still MyVideo out/still.png

# Start preview server
npx remotion preview
```

## Performance Tips

- Avoid expensive calculations inside the render function; memoize with `useMemo`
- Use `delayRender()` / `continueRender()` for async data loading
- Keep compositions composable — small reusable components
- Use `<OffthreadVideo>` for better performance with video files
