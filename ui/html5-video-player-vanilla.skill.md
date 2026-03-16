---
name: html5-video-player-vanilla
description: "Lightweight vanilla HTML/CSS/JS video player for static pages — play/pause, seek, mute, fullscreen, progress bar — zero dependencies, no React"
category: ui
difficulty: beginner
tags: [video, player, html5, media, vanilla-js, static-page, zero-dependency]
stack: [html, css, javascript]
---

# Vanilla HTML5 Video Player

You are a media UI expert. When asked to add video playback to a **static HTML page** (not React), build a custom lightweight player using the native `<video>` element with vanilla JS — no frameworks or dependencies.

## When to Use

- Static HTML pages (e.g., portfolio sites, landing pages, single-file apps)
- Pages that don't use React/Vue/etc.
- For React apps, use the `html5-video-player` skill instead

## CSS Styles

Add these styles inside `<style>` or your stylesheet:

```css
/* ── Video Showcase ── */
.video-showcase {
  position: relative;
  padding: 4rem 1.5rem;
}
.video-showcase::before {
  content: '';
  position: absolute; inset: 0;
  background: radial-gradient(ellipse at center, rgba(99,102,241,.08), transparent 70%);
  pointer-events: none;
}
.video-showcase-inner {
  position: relative;
  max-width: 900px;
  margin: 0 auto;
}
.video-showcase-header {
  text-align: center;
  margin-bottom: 2rem;
}
.video-badge {
  display: inline-flex; align-items: center; gap: .4rem;
  padding: .35rem 1rem; border-radius: 9999px;
  border: 1px solid rgba(99,102,241,.3);
  background: rgba(99,102,241,.1);
  color: #a5b4fc; font-size: .72rem; font-weight: 500;
  margin-bottom: 1rem;
}
.video-wrap {
  position: relative;
  border-radius: 1rem;
  overflow: hidden;
  border: 1px solid rgba(99,102,241,.2);
  box-shadow: 0 25px 50px rgba(99,102,241,.1);
  background: #000;
  aspect-ratio: 16/9;
}
.video-wrap video {
  width: 100%; height: 100%;
  object-fit: contain;
  display: block;
}
.video-play-overlay {
  position: absolute; inset: 0;
  display: flex; align-items: center; justify-content: center;
  background: rgba(0,0,0,.4);
  cursor: pointer;
  transition: opacity .2s;
}
.video-play-overlay.hidden { opacity: 0; pointer-events: none; }
.video-play-btn {
  width: 80px; height: 80px;
  border-radius: 50%;
  background: linear-gradient(135deg, #00c6ff, #6c63ff);
  display: flex; align-items: center; justify-content: center;
  box-shadow: 0 20px 40px rgba(0,198,255,.3);
  transition: transform .2s;
}
.video-play-btn:hover { transform: scale(1.1); }
.video-controls {
  position: absolute; bottom: 0; left: 0; right: 0;
  padding: 1rem;
  background: linear-gradient(to top, rgba(0,0,0,.8), transparent);
  opacity: 0; transition: opacity .2s;
}
.video-wrap:hover .video-controls { opacity: 1; }
.video-progress {
  width: 100%; height: 6px;
  background: rgba(255,255,255,.2);
  border-radius: 3px; cursor: pointer;
  margin-bottom: .75rem;
}
.video-progress-fill {
  height: 100%;
  background: linear-gradient(90deg, #00c6ff, #6c63ff);
  border-radius: 3px;
  transition: width .1s linear;
}
.video-controls-row {
  display: flex; align-items: center; justify-content: space-between;
}
.video-controls-row button {
  background: none; border: none; color: #fff;
  cursor: pointer; padding: 4px;
  transition: color .2s;
}
.video-controls-row button:hover { color: #00c6ff; }
.video-time {
  font-size: .72rem; color: #cbd5e1;
  font-family: monospace;
  margin-left: .5rem;
}
.video-caption {
  text-align: center; color: #8892a4;
  font-size: .72rem; margin-top: 1rem;
}
```

## HTML Structure

```html
<section class="video-showcase">
  <div class="video-showcase-inner">
    <div class="video-showcase-header">
      <div class="video-badge">
        <svg width="12" height="12" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><polygon points="5 3 19 12 5 21 5 3"/></svg>
        2 Minute Tour
      </div>
      <h2>See It In <span style="background:linear-gradient(135deg,#00c6ff,#6c63ff);-webkit-background-clip:text;-webkit-text-fill-color:transparent">Action</span></h2>
    </div>
    <div class="video-wrap" id="videoWrap">
      <video id="tourVideo" preload="metadata" playsinline
        src="VIDEO_URL_HERE">
      </video>
      <div class="video-play-overlay" id="videoOverlay" onclick="toggleVideo()">
        <div class="video-play-btn">
          <svg width="32" height="32" viewBox="0 0 24 24" fill="white"><polygon points="5 3 19 12 5 21 5 3"/></svg>
        </div>
      </div>
      <div class="video-controls">
        <div class="video-progress" onclick="seekVideo(event)">
          <div class="video-progress-fill" id="videoProgressFill" style="width:0%"></div>
        </div>
        <div class="video-controls-row">
          <div style="display:flex;align-items:center">
            <button onclick="toggleVideo()" id="videoPauseBtn">
              <svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><polygon points="5 3 19 12 5 21 5 3"/></svg>
            </button>
            <button onclick="toggleMute()" id="videoMuteBtn">
              <svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><polygon points="11 5 6 9 2 9 2 15 6 15 11 19 11 5"/><path d="M19.07 4.93a10 10 0 0 1 0 14.14"/><path d="M15.54 8.46a5 5 0 0 1 0 7.07"/></svg>
            </button>
            <span class="video-time" id="videoTime">0:00 / 0:00</span>
          </div>
          <button onclick="document.getElementById('tourVideo').requestFullscreen()">
            <svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><polyline points="15 3 21 3 21 9"/><polyline points="9 21 3 21 3 15"/><line x1="21" y1="3" x2="14" y2="10"/><line x1="3" y1="21" x2="10" y2="14"/></svg>
          </button>
        </div>
      </div>
    </div>
    <p class="video-caption">Caption text here</p>
  </div>
</section>
```

## JavaScript

Place this `<script>` after the video HTML:

```html
<script>
(function(){
  var vid = document.getElementById('tourVideo');
  var overlay = document.getElementById('videoOverlay');
  var fill = document.getElementById('videoProgressFill');
  var timeEl = document.getElementById('videoTime');
  var pauseBtn = document.getElementById('videoPauseBtn');
  var muteBtn = document.getElementById('videoMuteBtn');
  var isPlaying = false;

  function fmt(s) {
    var m = Math.floor(s / 60);
    var sec = Math.floor(s % 60);
    return m + ':' + (sec < 10 ? '0' : '') + sec;
  }

  vid.addEventListener('timeupdate', function() {
    if (vid.duration) {
      fill.style.width = (vid.currentTime / vid.duration * 100) + '%';
      timeEl.textContent = fmt(vid.currentTime) + ' / ' + fmt(vid.duration);
    }
  });
  vid.addEventListener('ended', function() {
    isPlaying = false;
    overlay.classList.remove('hidden');
    pauseBtn.innerHTML = '<svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><polygon points="5 3 19 12 5 21 5 3"/></svg>';
  });
  vid.addEventListener('loadedmetadata', function() {
    timeEl.textContent = '0:00 / ' + fmt(vid.duration);
  });

  window.toggleVideo = function() {
    if (isPlaying) {
      vid.pause();
      overlay.classList.remove('hidden');
      pauseBtn.innerHTML = '<svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><polygon points="5 3 19 12 5 21 5 3"/></svg>';
    } else {
      vid.play();
      overlay.classList.add('hidden');
      pauseBtn.innerHTML = '<svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><rect x="6" y="4" width="4" height="16"/><rect x="14" y="4" width="4" height="16"/></svg>';
    }
    isPlaying = !isPlaying;
  };

  window.toggleMute = function() {
    vid.muted = !vid.muted;
    muteBtn.innerHTML = vid.muted
      ? '<svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><polygon points="11 5 6 9 2 9 2 15 6 15 11 19 11 5"/><line x1="23" y1="9" x2="17" y2="15"/><line x1="17" y1="9" x2="23" y2="15"/></svg>'
      : '<svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><polygon points="11 5 6 9 2 9 2 15 6 15 11 19 11 5"/><path d="M19.07 4.93a10 10 0 0 1 0 14.14"/><path d="M15.54 8.46a5 5 0 0 1 0 7.07"/></svg>';
  };

  window.seekVideo = function(e) {
    var rect = e.currentTarget.getBoundingClientRect();
    var pct = (e.clientX - rect.left) / rect.width;
    vid.currentTime = pct * vid.duration;
  };
})();
</script>
```

## Key Design Decisions

- **Zero dependencies**: Native `<video>` + vanilla JS — no React, no build step
- **IIFE pattern**: All state scoped to avoid global pollution (except click handlers exposed on `window`)
- **`playsinline`**: Required for iOS Safari
- **`preload="metadata"`**: Loads duration without downloading full video
- **`object-fit: contain`**: Preserves aspect ratio
- **Hover controls**: Controls bar appears on hover via CSS only
- **Gradient progress bar**: Matches app design system (cyan → indigo)
- **Click-to-seek**: Progress bar is clickable for precise seeking
- **SVG icons**: Inline SVGs — no icon library needed

## Video Hosting

Host MP4s on S3 + CloudFront for fast global delivery:
```bash
aws s3 cp video.mp4 s3://bucket/videos/video.mp4 --content-type "video/mp4" --cache-control "public, max-age=31536000"
aws cloudfront create-invalidation --distribution-id DIST_ID --paths "/videos/*"
```
