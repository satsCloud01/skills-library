---
name: product-tour-video
description: "Generate a Remotion-based animated product tour video from app tour data — category transitions, feature slides with capabilities, orbital logo outro, epic background music — navy/gold enterprise aesthetic"
category: ui
difficulty: advanced
tags: [video, remotion, tour, product-launch, animation, cinematic, enterprise, branding]
stack: [react, typescript, remotion]
---

# Detailed Product Tour Video

You are a video production engineer specializing in Remotion-based product tour videos. When asked to create a product tour or demo video, build a complete Remotion project that transforms app/feature data into a cinematic animated video with category transitions, feature slides, and a branded outro.

## When to Use

- Creating product demo/tour videos from structured app data
- Animated portfolio showcases with per-feature slides
- Product launch videos with cinematic transitions
- Any video that walks through features, capabilities, or solutions

## Project Structure

```
project-root/
├── package.json
├── tsconfig.json
├── public/
│   └── bgm.mp3              # Background music (royalty-free)
└── src/
    ├── index.tsx             # registerRoot entry
    ├── Root.tsx              # Composition definition
    ├── ProductVideo.tsx      # Main sequence builder
    ├── tourData.ts           # Structured tour data
    └── scenes/
        ├── IntroScene.tsx    # Opening with logo, title, stats
        ├── CategoryTransition.tsx  # Category divider slides
        ├── FeatureScene.tsx  # Per-feature capability slides
        └── OutroScene.tsx    # Closing with animated logo + CTA
```

## Tour Data Schema

Each feature/app in the tour must follow this interface:

```typescript
export interface TourStep {
  icon: string;           // Emoji icon for the feature
  title: string;          // Feature/app name
  subtitle: string;       // Short descriptor
  gradient: string;       // CSS gradient for accent
  url: string | null;     // Live URL (null if not deployed)
  description: string;    // Two-liner description shown under title
  capabilities: string[]; // 4-5 top capabilities as bullet points
  category: string;       // Category grouping for transitions
}

export interface Category {
  name: string;
  color: string;          // Hex accent color
  icon: string;           // Emoji icon
}
```

## Timing Configuration

```typescript
const INTRO_DURATION = 6 * 30;       // 6 seconds at 30fps
const FEATURE_DURATION = 5 * 30;     // 5 seconds per feature
const TRANSITION_DURATION = 2 * 30;  // 2 seconds per category transition
const OUTRO_DURATION = 10 * 30;      // 10 seconds for logo outro
```

Total duration formula: `INTRO + (num_categories × TRANSITION) + (num_features × FEATURE) + OUTRO`

## Scene Design Patterns

### Background System (consistent across all scenes)

Every scene uses this layered background system for a premium cinematic feel:

1. **Base**: Dark navy gradient (`#060d18` → `#0a1628` → `#111d35`)
2. **Grid mesh**: Subtle gold grid lines with animated opacity (`rgba(200,169,110,0.03-0.07)`)
3. **Glow blobs**: Radial gradients with blur (accent-colored, slowly rotating)
4. **Floating particles**: Small dots that drift upward over the scene duration
5. **Orbital rings** (intro/outro): SVG ellipses with CSS rotation animation

```tsx
// Grid with animated pulse
<div style={{
  position: "absolute", inset: 0,
  backgroundImage: `
    linear-gradient(rgba(200,169,110,${gridPulse}) 1px, transparent 1px),
    linear-gradient(90deg, rgba(200,169,110,${gridPulse}) 1px, transparent 1px)
  `,
  backgroundSize: "60px 60px",
}} />

// Accent glow blob
<div style={{
  position: "absolute", top: -150, right: -150,
  width: 600, height: 600, borderRadius: "50%",
  background: `radial-gradient(circle, ${accentColor}12 0%, transparent 65%)`,
  filter: "blur(80px)",
  transform: `rotate(${meshRotate}deg)`,
}} />

// Floating particles
{[...Array(12)].map((_, i) => (
  <div key={i} style={{
    position: "absolute",
    width: 2 + (i % 3), height: 2 + (i % 3),
    borderRadius: "50%",
    background: i % 2 === 0 ? "#c8a96e" : "rgba(99,102,241,0.6)",
    left: `${10 + i * 7.5}%`,
    top: `${70 + Math.sin(i * 0.8) * 20}%`,
    opacity: interpolate(frame, [0, 150], [0.15, 0.6], { extrapolateRight: "clamp" }),
    transform: `translateY(${particleY * (0.3 + (i % 4) * 0.2)}px)`,
  }} />
))}
```

### IntroScene

- Spring-animated logo mark (circle with monogram letter)
- Large gradient title text (Georgia serif)
- Subtitle with brand tagline (italic gold)
- Stats row (4 key metrics in bordered cards)
- Background: orbital SVG rings + grid + particles

### CategoryTransition

- Large category emoji icon (90px)
- Category name in serif font
- Animated gold line expanding from center
- Pulsing concentric rings in category accent color
- "SATSZONE.LINK" subtle label below

### FeatureScene (the core slide)

Layout (top to bottom):
1. **Top bar**: Category badge (pill) + step counter + progress bar
2. **Title row**: Icon (76px rounded square) + Title (46px serif) + Subtitle
3. **Description**: Two-liner in 24px with gold left-border accent
4. **"Top Capabilities" header**: Uppercase gold label
5. **Capability cards**: 5 numbered items, staggered slide-in animation (12-frame stagger)
6. **URL indicator**: Green dot + monospace URL (if available)

Animation timing per slide (at 30fps):
- Frame 0-18: Header slides down
- Frame 3+: Title springs in
- Frame 15-30: Description fades in
- Frame 35+: Capabilities stagger in (12 frames apart)
- Frame 95+: URL indicator fades in

```tsx
// Capability stagger pattern
const capBaseDelay = 35;
const capStagger = 12;

{step.capabilities.map((cap, i) => {
  const delay = capBaseDelay + i * capStagger;
  const capOpacity = interpolate(frame, [delay, delay + 12], [0, 1], { extrapolateRight: "clamp" });
  const capX = interpolate(frame, [delay, delay + 12], [30, 0], { extrapolateRight: "clamp" });
  // ... render numbered card
})}
```

### OutroScene

- Animated orbital logo (SVG with rotating ellipses, pillar labels staggering in)
- Large "BRAND NAME" text with spring animation
- Tagline in italic gold
- Stats row
- CTA button with gold gradient and box-shadow
- Extended duration (10s) for dramatic effect

The orbital logo SVG must set `transformOrigin` on rotating `<g>` elements to match the ellipse center:
```tsx
<g style={{
  transformOrigin: "120px 88px",  // MUST match ellipse cx, cy
  transform: `rotate(${ring1Rotate}deg)`,
}}>
```

## Background Music

- Use royalty-free music (Pixabay recommended — no attribution needed)
- Loop if track is shorter than video: `ffmpeg -stream_loop 1 -i bgm.mp3 -t DURATION -af "afade=t=in:d=2,afade=t=out:st=DURATION-4:d=4" bgm_looped.mp3`
- Place in `public/bgm.mp3`
- Add to composition with fade in/out:

```tsx
const BackgroundMusic: React.FC = () => {
  const frame = useCurrentFrame();
  const volume = interpolate(
    frame,
    [0, 90, TOTAL_FRAMES - 120, TOTAL_FRAMES],
    [0, 1, 1, 0],
    { extrapolateLeft: "clamp", extrapolateRight: "clamp" }
  );
  return <Audio src={staticFile("bgm.mp3")} volume={volume} />;
};
```

## Sequence Builder

The main composition auto-builds from tour data:

```typescript
function buildSequence(): SequenceItem[] {
  const items: SequenceItem[] = [{ type: "intro", duration: INTRO_DURATION }];
  let lastCategory = "";
  TOUR_STEPS.forEach((step, i) => {
    if (step.category !== lastCategory) {
      items.push({ type: "transition", duration: TRANSITION_DURATION, category: step.category });
      lastCategory = step.category;
    }
    items.push({ type: "feature", duration: FEATURE_DURATION, stepIndex: i });
  });
  items.push({ type: "outro", duration: OUTRO_DURATION });
  return items;
}
```

## Color Palette

| Role | Hex | Usage |
|------|-----|-------|
| Base Navy | `#060d18` | Darkest background |
| Navy | `#0a1628` | Primary background |
| Mid Navy | `#111d35` | Gradient stop |
| Gold | `#c8a96e` | Strokes, accents, text |
| Light Gold | `#e8c98e` | Highlights, gradient stops |
| Deep Gold | `#a8894e` | Depth gradient stops |
| Muted | `#8a8070` | Secondary text |
| Text | `#e8e0d4` | Primary text on dark |

## Commands

```bash
# Preview in Remotion Studio
npm run studio

# Render to MP4
npm run render
# or: npx remotion render src/index.tsx CompositionId out/video.mp4

# Render specific frame range (for testing)
npx remotion render src/index.tsx CompositionId --frames=0-300 out/test.mp4
```

## Rendering for Production

When rendering the final video for deployment:

```bash
npx remotion render src/index.tsx CompositionId out/video.mp4 \
  --codec=h264 \
  --image-format=jpeg \
  --quality=80
```

Then upload: `aws s3 cp out/video.mp4 s3://bucket/videos/tour.mp4 --content-type "video/mp4"`

## Customization Checklist

1. Update `tourData.ts` with your app/feature data
2. Adjust category colors to match your brand
3. Replace `public/bgm.mp3` with your preferred music
4. Update intro title, tagline, and stats
5. Update outro logo SVG, brand name, tagline, and CTA URL
6. Adjust timing constants if slides feel too fast/slow
7. Update `Root.tsx` composition ID and dimensions (default: 1920×1080 at 30fps)
