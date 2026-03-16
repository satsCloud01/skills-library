---
name: html5-video-player
description: "Lightweight custom HTML5 video player component with play/pause, seek, mute, fullscreen, and progress bar — zero external dependencies"
category: ui
difficulty: intermediate
tags: [video, player, html5, media, lightweight, zero-dependency]
stack: [react-18, tailwind]
---

# HTML5 Video Player Component

You are a media UI expert. When asked to add video playback to a React app, build a custom lightweight player using the native HTML5 `<video>` element — no external libraries needed.

## Component: VideoPlayer

### Props
```jsx
// Required
src        // string — URL to MP4/WebM video
// Optional
className  // string — additional CSS classes
onClose    // function — if provided, renders a close button
poster     // string — URL to poster/thumbnail image
```

### Features to implement
1. **Play/Pause** — click video or button to toggle
2. **Seek** — clickable progress bar
3. **Time display** — current / duration in m:ss format
4. **Mute/Unmute** — toggle audio
5. **Fullscreen** — native fullscreen API
6. **Play overlay** — large centered play button when paused

### Implementation Pattern

```jsx
import { useState, useRef } from 'react';
import { Play, Pause, Volume2, VolumeX, Maximize } from 'lucide-react';

function VideoPlayer({ src, className = '', onClose, poster }) {
  const videoRef = useRef(null);
  const [playing, setPlaying] = useState(false);
  const [muted, setMuted] = useState(false);
  const [progress, setProgress] = useState(0);
  const [duration, setDuration] = useState(0);

  const toggle = () => {
    if (!videoRef.current) return;
    if (playing) videoRef.current.pause();
    else videoRef.current.play();
    setPlaying(!playing);
  };

  const fmt = (s) => {
    const m = Math.floor(s / 60);
    const sec = Math.floor(s % 60);
    return `${m}:${sec.toString().padStart(2, '0')}`;
  };

  return (
    <div className={`relative group rounded-2xl overflow-hidden bg-black ${className}`}>
      <video
        ref={videoRef}
        src={src}
        poster={poster}
        className="w-full h-full object-contain"
        muted={muted}
        onTimeUpdate={() => setProgress(videoRef.current?.currentTime || 0)}
        onLoadedMetadata={() => setDuration(videoRef.current?.duration || 0)}
        onEnded={() => setPlaying(false)}
        onClick={toggle}
        playsInline
        preload="metadata"
      />

      {/* Large play button overlay when paused */}
      {!playing && (
        <div className="absolute inset-0 flex items-center justify-center bg-black/40 cursor-pointer" onClick={toggle}>
          <div className="w-20 h-20 rounded-full bg-gradient-to-br from-cyan-500 to-indigo-500 flex items-center justify-center shadow-2xl hover:scale-110 transition-transform">
            <Play className="w-8 h-8 text-white ml-1" fill="white" />
          </div>
        </div>
      )}

      {/* Controls bar — visible on hover */}
      <div className="absolute bottom-0 left-0 right-0 bg-gradient-to-t from-black/80 to-transparent p-4 opacity-0 group-hover:opacity-100 transition-opacity">
        {/* Seekable progress bar */}
        <div
          className="w-full h-1.5 bg-white/20 rounded-full mb-3 cursor-pointer"
          onClick={(e) => {
            const rect = e.currentTarget.getBoundingClientRect();
            const pct = (e.clientX - rect.left) / rect.width;
            if (videoRef.current) videoRef.current.currentTime = pct * duration;
          }}
        >
          <div
            className="h-full bg-gradient-to-r from-cyan-400 to-indigo-400 rounded-full transition-all"
            style={{ width: duration ? `${(progress / duration) * 100}%` : '0%' }}
          />
        </div>

        <div className="flex items-center justify-between">
          <div className="flex items-center gap-3">
            <button onClick={toggle} className="text-white hover:text-cyan-400">
              {playing ? <Pause className="w-5 h-5" /> : <Play className="w-5 h-5" />}
            </button>
            <button onClick={() => setMuted(!muted)} className="text-white hover:text-cyan-400">
              {muted ? <VolumeX className="w-5 h-5" /> : <Volume2 className="w-5 h-5" />}
            </button>
            <span className="text-xs text-gray-300 font-mono">
              {fmt(progress)} / {fmt(duration)}
            </span>
          </div>
          <button
            onClick={() => videoRef.current?.requestFullscreen?.()}
            className="text-white hover:text-cyan-400"
          >
            <Maximize className="w-5 h-5" />
          </button>
        </div>
      </div>
    </div>
  );
}
```

### Video Modal Pattern

For modal/overlay video (e.g., triggered from a tour or button):

```jsx
function VideoModal({ src, onClose }) {
  return (
    <div className="fixed inset-0 z-[200] flex items-center justify-center bg-black/70 backdrop-blur-sm" onClick={onClose}>
      <div className="relative w-full max-w-4xl mx-4 rounded-2xl overflow-hidden shadow-2xl border border-indigo-500/30" onClick={e => e.stopPropagation()}>
        {/* Header bar */}
        <div className="flex items-center justify-between px-4 py-2.5 bg-gradient-to-r from-indigo-950 to-gray-900">
          <span className="text-sm font-semibold text-white">Video Title</span>
          <button onClick={onClose} className="text-gray-400 hover:text-white">
            <X className="w-5 h-5" />
          </button>
        </div>
        {/* Reuse VideoPlayer inside */}
        <VideoPlayer src={src} className="aspect-video" />
      </div>
    </div>
  );
}
```

### Usage in Landing Pages

Embed as a showcase section:

```jsx
<section className="py-16 px-6 bg-gray-950">
  <div className="max-w-5xl mx-auto text-center">
    <h2 className="text-3xl font-bold mb-8">See It In Action</h2>
    <div className="rounded-2xl border border-indigo-500/20 shadow-2xl overflow-hidden">
      <VideoPlayer src={VIDEO_URL} className="aspect-video" />
    </div>
  </div>
</section>
```

### Key Design Decisions
- **Zero dependencies**: Uses native `<video>` element — no react-player, vidstack, or plyr needed
- **playsInline**: Required for iOS Safari autoplay
- **preload="metadata"**: Loads duration/dimensions without downloading full video
- **object-contain**: Preserves aspect ratio without cropping
- **group-hover controls**: Controls appear on hover, stay hidden otherwise
- **Gradient progress bar**: Matches app's design system (cyan → indigo)
- **Click-to-seek**: Progress bar is clickable for precise seeking

### Video Hosting
Host MP4s on S3 + CloudFront for fast global delivery:
```bash
aws s3 cp video.mp4 s3://bucket/videos/video.mp4 --content-type "video/mp4"
aws cloudfront create-invalidation --distribution-id DIST_ID --paths "/videos/*"
```
