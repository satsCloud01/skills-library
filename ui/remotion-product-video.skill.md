---
name: remotion-product-video
description: "Creates polished product launch videos using Remotion (React-based video framework) with animated scenes, category transitions, background music, and professional typography"
category: ui
difficulty: advanced
tags: [remotion, video, product-launch, animation, react, marketing]
stack: [react-18, remotion-4, typescript]
---

# Remotion Product Video Generator

You are a video production expert using Remotion (React-based programmatic video framework). When asked to create a product video, demo video, or launch video, follow this skill exactly.

## Project Setup

1. Create the project directory and initialize:

```bash
mkdir <project-name>-video && cd <project-name>-video
npm init -y
npm install remotion @remotion/cli @remotion/player react@18 react-dom@18 typescript @types/react@18 @types/react-dom@18
mkdir -p src/scenes public
```

2. Create `remotion.config.js`:

```js
import { Config } from "@remotion/cli/config";
Config.setVideoImageFormat("jpeg");
Config.setOverwriteOutput(true);
```

3. Create `tsconfig.json`:

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "ES2022",
    "moduleResolution": "bundler",
    "jsx": "react-jsx",
    "strict": false,
    "esModuleInterop": true,
    "skipLibCheck": true
  },
  "include": ["src/**/*"]
}
```

4. Add scripts to `package.json`:

```json
{
  "scripts": {
    "studio": "remotion studio src/index.tsx",
    "render": "remotion render src/index.tsx <CompositionId> out/video.mp4",
    "preview": "remotion preview src/index.tsx"
  }
}
```

## Architecture

Every product video follows this scene structure:

```
IntroScene → [CategoryTransition → FeatureScene]... → OutroScene
```

### Entry Point (`src/index.tsx`)

```tsx
import { registerRoot } from "remotion";
import { RemotionRoot } from "./Root";
registerRoot(RemotionRoot);
```

### Root Composition (`src/Root.tsx`)

```tsx
import { Composition } from "remotion";
import { ProductVideo } from "./ProductVideo";

export const RemotionRoot: React.FC = () => (
  <Composition
    id="ProductVideo"
    component={ProductVideo}
    durationInFrames={CALCULATED_TOTAL}
    fps={30}
    width={1920}
    height={1080}
  />
);
```

### Data Layer (`src/tourData.ts`)

Extract the product's feature content into a typed data structure:

```ts
export interface TourStep {
  title: string;
  description: string;
  page: string;        // route or screen name
  example: string;     // "try this" actionable text
  tip: string;         // pro tip / insight
  icon: string;        // emoji icon
  category: string;    // grouping category
}

export interface Category {
  name: string;
  color: string;       // hex accent color
}
```

- Source content from the app's existing tour, onboarding, or feature list
- Group features into 4-8 categories for visual rhythm
- Each feature needs: title, 1-line description, example action, pro tip

### Main Composition (`src/ProductVideo.tsx`)

Build a sequence dynamically from the data:

```tsx
import { AbsoluteFill, Audio, Sequence, interpolate, useCurrentFrame, staticFile } from "remotion";

// Timing constants (frames at 30fps)
const INTRO_DURATION = 150;      // 5 seconds
const FEATURE_DURATION = 150;    // 5 seconds per feature
const TRANSITION_DURATION = 60;  // 2 seconds per category
const OUTRO_DURATION = 150;      // 5 seconds

// Build sequence: intro → [transition + features per category] → outro
// Calculate TOTAL_FRAMES from the built sequence

// Background music with fade in/out
const BackgroundMusic: React.FC = () => {
  const frame = useCurrentFrame();
  const volume = interpolate(
    frame,
    [0, 60, TOTAL_FRAMES - 90, TOTAL_FRAMES],
    [0, 0.35, 0.35, 0],
    { extrapolateLeft: "clamp", extrapolateRight: "clamp" }
  );
  return <Audio src={staticFile("bgm.mp3")} volume={volume} />;
};
```

## Scene Templates

### IntroScene

Purpose: Brand reveal with animated title and tagline.

Key elements:
- **Background**: Dark gradient (#0f0a1e → #1e1b4b → #312e81) with animated radial orbs
- **Grid overlay**: Subtle line grid for tech feel
- **Logo**: Spring-animated icon in gradient rounded square
- **Title**: Large gradient text (white → light indigo → cyan), `fontWeight: 800`, `letterSpacing: -2`
- **Subtitle**: Fades in at ~1s, indigo-tinted (#a5b4fc)
- **Tagline**: Slides up + fades in at ~2s, cyan-tinted (#67e8f9), shows key stats

Animations:
```tsx
const titleScale = spring({ frame, fps, config: { damping: 12 } });
const subtitleOpacity = interpolate(frame, [30, 50], [0, 1], { extrapolateRight: "clamp" });
const taglineY = interpolate(frame, [60, 80], [30, 0], { extrapolateRight: "clamp" });
```

### CategoryTransition

Purpose: Full-screen category divider between feature groups.

Key elements:
- **Background**: Dark with radial glow in category accent color
- **Icon**: Large emoji (80px)
- **Category name**: Bold white text (52px)
- **Underline**: Animated gradient line that expands from center

Animations:
```tsx
const scale = spring({ frame, fps, config: { damping: 10 } });
const lineWidth = interpolate(frame, [10, 40], [0, 400], { extrapolateRight: "clamp" });
```

### FeatureScene

Purpose: Showcase individual feature with description and examples.

Layout (top to bottom):
1. **Top bar**: Category pill (accent color bg) + step counter + progress bar
2. **Icon + Title row**: Emoji in bordered rounded square + feature title (48px bold white)
3. **Description**: Gray text (26px, #cbd5e1), max 1200px width
4. **Two-column cards**:
   - Left: "Try This" box — indigo tones, play icon, example text
   - Right: "Pro Tip" box — cyan tones, lightbulb icon, tip text
5. **Bottom**: Route/URL indicator with green dot

Animation cascade (staggered entrance):
```tsx
// Header slides down: frames 0-20
// Title springs in: frames 5+
// Description fades up: frames 20-40
// Example card slides in from right: frames 45-65
// Tip card slides in from right: frames 70-90
```

Color system per card:
- Try This: `rgba(99,102,241,0.08)` bg, `rgba(99,102,241,0.2)` border, `#a5b4fc`/`#c7d2fe` text
- Pro Tip: `rgba(6,182,212,0.08)` bg, `rgba(6,182,212,0.2)` border, `#67e8f9`/`#a5f3fc` text

### OutroScene

Purpose: Call to action with stats summary.

Key elements:
- **Headline**: Gradient text "Ready to [action]?"
- **Subtitle**: Product positioning line
- **Stats row**: 4 cards with key metrics (modules, features, pages, endpoints)
- **CTA button**: Gradient pill (indigo → cyan) with product URL
- **Footer**: "Built with [tech stack]"

Animations:
```tsx
const scale = spring({ frame, fps, config: { damping: 12 } });
const statsOpacity = interpolate(frame, [30, 50], [0, 1], { extrapolateRight: "clamp" });
const ctaScale = spring({ frame: frame - 60, fps, config: { damping: 10 } });
```

## Background Music

Place an MP3 in `public/bgm.mp3`. Good sources for royalty-free corporate/tech music:
- Free Music Archive (CC-BY/CC0)
- Pixabay Music (Pixabay License)

Audio configuration:
- Volume: 0.35 (subtle, not overwhelming)
- Fade in: 2 seconds (60 frames)
- Fade out: 3 seconds (90 frames)
- Use `staticFile("bgm.mp3")` with Remotion's `<Audio>` component

## Design System

### Colors
| Role | Value |
|------|-------|
| Background | `#0f0a1e` |
| Surface | `#1a1640` |
| Primary | `#6366f1` (indigo) |
| Accent | `#06b6d4` (cyan) |
| Text primary | `#ffffff` |
| Text secondary | `#cbd5e1` |
| Text muted | `#94a3b8` |

### Typography
- All text uses system font stack: `-apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif`
- Titles: 48-72px, weight 800, letterSpacing -1 to -2
- Body: 22-26px, weight 400, lineHeight 1.6
- Labels: 16px, weight 700, uppercase, letterSpacing 1

### Animation Principles
- Use `spring()` for scale/entrance (damping: 10-12)
- Use `interpolate()` for opacity and slide (20-30 frame transitions)
- Stagger elements by 15-25 frames for cascade effect
- Never animate everything at once — build the scene progressively

## Duration Calculation

```
Total = INTRO + Σ(TRANSITION per unique category + FEATURE per step) + OUTRO
```

For a 16-feature app with 8 categories:
- Intro: 150 frames (5s)
- 8 transitions × 60 = 480 frames (16s)
- 16 features × 150 = 2400 frames (80s)
- Outro: 150 frames (5s)
- **Total: 3180 frames (~106s)**

Adjust `FEATURE_DURATION` to control video length. For shorter videos, use 90-120 frames (3-4s) per feature.

## Rendering

```bash
# Preview in browser
npm run studio

# Render to MP4 (requires ffmpeg for audio)
npm run render

# Render specific frame range
npx remotion render src/index.tsx ProductVideo out/clip.mp4 --frames=0-300
```

## Adapting to Any Product

1. **Extract content**: Find the app's tour/onboarding/feature list component
2. **Map to TourStep[]**: Title, description, example, tip, category for each feature
3. **Define categories**: 4-8 logical groupings with accent colors
4. **Update branding**: Logo emoji, app name, tagline, URL, tech stack in Intro/Outro
5. **Adjust timing**: More features = consider shorter FEATURE_DURATION
6. **Choose music**: Match the product's energy — upbeat for SaaS, ambient for enterprise
