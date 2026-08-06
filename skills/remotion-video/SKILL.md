---
name: remotion-video
version: 1.0.0
description: "When the user wants to create marketing videos programmatically with code using Remotion and React — product demos, social media ads, feature announcements, animated explainers, testimonial videos, stat/metric reveals, or launch teasers. Also use when the user mentions 'Remotion,' 'video in React,' 'programmatic video,' 'code-based video,' 'motion graphics,' 'animated ad,' 'render a video,' or 'video from a template.' For static social posts and copy, see social-content. For ad strategy and targeting, see paid-ads."
---

# Remotion Video

You are an expert in programmatic video and motion design for marketing. Your goal is to help create on-brand marketing videos as code with [Remotion](https://www.remotion.dev) — a framework for making real MP4/WebM videos using React. Because the video is code, it is version-controlled, data-driven, and re-renderable at scale (one template → hundreds of variants).

## When Remotion Is the Right Tool

**Use Remotion when the video should be:**
- **Templated and repeatable** — one composition rendered for many products, plans, or customers
- **Data-driven** — charts, metrics, leaderboards, or personalized numbers pulled from an API or CSV
- **Programmatically varied** — 20 ad sizes/aspect ratios from one source, or A/B variants of a hook
- **Version-controlled** — the video lives in the repo next to the product

**Reach for something else when:**
- You need photorealistic footage, actors, or AI-generated scenes → use a generative video tool
- It's a one-off edit of existing footage → use a video editor (CapCut, Premiere)
- The deliverable is a static image → use the design/image tools

## Before Creating

**Check for product marketing context first:** If `.claude/product-marketing-context.md` exists, read it for brand voice, colors, audience, and positioning before asking questions.

Then gather (ask only for what's missing):

1. **Goal & placement** — What's the video for? (paid social ad, organic post, product page hero, launch teaser, email GIF). Where will it run? This sets aspect ratio and duration.
2. **Message** — The single takeaway. One video, one idea. What's the hook (first 1-2 seconds) and the CTA (last frame)?
3. **Brand** — Colors (hex), fonts, logo, tone. Any motion do's/don'ts.
4. **Assets** — Logo files, screenshots, product footage, headshots, data source.
5. **Specs** — Aspect ratio, duration, whether captions/sound are needed (most social autoplay is muted → captions are mandatory).

## Platform Specs Quick Reference

| Placement | Aspect | Resolution | Duration | fps |
|-----------|--------|------------|----------|-----|
| TikTok / Reels / Shorts / Stories | 9:16 | 1080×1920 | 5–30s | 30 |
| Feed (Meta, LinkedIn) | 1:1 or 4:5 | 1080×1080 / 1080×1350 | 5–15s | 30 |
| YouTube / landing hero | 16:9 | 1920×1080 | 6–30s | 30 |
| Email / inline GIF | 16:9 or 1:1 | 800×450 / 600×600 | 2–6s (loop) | 24–30 |

**Rule of thumb:** hook in the first 1.5s, brand within 3s, CTA held for the final 2s. Design muted-first — the story must land without sound.

## Project Setup

Scaffold a new Remotion project, or add a composition to an existing one:

```bash
# New standalone project
npm create video@latest -- --blank my-marketing-videos
cd my-marketing-videos
npm run dev            # opens Remotion Studio to preview
```

Key files:
- `src/Root.tsx` — registers every `<Composition>` (id, dimensions, fps, durationInFrames, default props)
- `src/<Name>.tsx` — the React component that renders each frame
- `remotion.config.ts` — render defaults (codec, image format, concurrency)

Register a composition in `src/Root.tsx`:

```tsx
import { Composition } from "remotion";
import { AdReel } from "./AdReel";

export const RemotionRoot: React.FC = () => (
  <Composition
    id="AdReel"
    component={AdReel}
    durationInFrames={9 * 30}   // 9 seconds at 30fps
    fps={30}
    width={1080}
    height={1920}               // 9:16 vertical
    defaultProps={{ headline: "Ship faster", cta: "Start free" }}
  />
);
```

## Core Concepts (What You Actually Need)

Remotion renders one React frame at a time. Animation = mapping the current frame number to style values.

- **`useCurrentFrame()`** — the frame being rendered (0-indexed). Everything animates off this.
- **`useVideoConfig()`** — `{ fps, width, height, durationInFrames }`.
- **`interpolate(frame, [inFrame, outFrame], [from, to], { extrapolateLeft: "clamp", extrapolateRight: "clamp" })`** — the workhorse for fades, slides, and progress. Always clamp so values don't overshoot outside the range.
- **`spring({ frame, fps, config })`** — natural, physics-based motion for entrances (logos, cards popping in).
- **`<Sequence from={f} durationInFrames={n}>`** — time-shifts children so a block starts at frame `f`. This is how you build scenes on a timeline.
- **`<Series>`** — chains sequences back-to-back without hand-computing offsets.
- **Media**: `<Img>`, `<Video>`/`<OffthreadVideo>`, `<Audio>`, and `staticFile("logo.png")` for assets in `public/`.
- **`<AbsoluteFill>`** — full-frame layered container; the default building block for scenes and backgrounds.

Minimal animated headline:

```tsx
import { AbsoluteFill, interpolate, useCurrentFrame } from "remotion";

export const AdReel: React.FC<{ headline: string }> = ({ headline }) => {
  const frame = useCurrentFrame();
  const opacity = interpolate(frame, [0, 15], [0, 1], { extrapolateRight: "clamp" });
  const y = interpolate(frame, [0, 15], [40, 0], { extrapolateRight: "clamp" });
  return (
    <AbsoluteFill style={{ backgroundColor: "#0B0B0F", justifyContent: "center", alignItems: "center" }}>
      <h1 style={{ color: "white", fontSize: 96, opacity, transform: `translateY(${y}px)` }}>
        {headline}
      </h1>
    </AbsoluteFill>
  );
};
```

**For copy-paste marketing scene recipes** (hook → benefits → CTA ad, stat reveal, screenshot demo, captions, data-driven variants), see [references/recipes.md](references/recipes.md).

## Motion & Brand Guidelines

- **Ease, don't lerp linearly.** Prefer `spring()` or eased `interpolate` for anything the eye tracks. Linear motion feels robotic.
- **Stagger entrances.** Offset each element by 2-4 frames so scenes feel choreographed, not simultaneous.
- **Hold readable frames.** Text needs ~0.8-1.2s on screen to be read. Don't animate faster than comprehension.
- **Brand within 3 seconds.** Logo/product visible early — muted autoplay scrolls fast.
- **Captions always.** Burn in captions for social; assume no sound. Keep them high-contrast and in the safe zone (clear of platform UI along the bottom ~15% and right edge).
- **One idea per video.** If there are two messages, make two videos.
- **Respect brand tokens.** Pull colors, fonts, and spacing from the product marketing context; don't invent a palette.

## Rendering & Export

```bash
# Render an MP4
npx remotion render AdReel out/ad-reel.mp4

# Override props at render time (great for variants)
npx remotion render AdReel out/ad-plan-pro.mp4 --props='{"headline":"Go Pro","cta":"Upgrade"}'

# Export a looping GIF for email
npx remotion render AdReel out/teaser.gif --codec=gif

# Still frame (e.g. a thumbnail) at frame 60
npx remotion still AdReel out/thumb.png --frame=60
```

**Batch/personalized rendering** (many variants from data): drive `--props` from a CSV/JSON in a loop, or use `renderMedia()` from the `@remotion/renderer` Node API for full programmatic control. See [references/recipes.md](references/recipes.md) for the batch pattern.

## Deeper Remotion Help

For advanced Remotion APIs, component references, and up-to-date best practices, install Remotion's official Claude Code plugin:

```bash
/plugin marketplace add remotion-dev/claude-code-plugin
/plugin install remotion@remotion
```

That plugin ships Remotion's own agent skills; this skill focuses on applying Remotion to **marketing** outcomes.

## Related Skills

- **social-content** — the messaging, hooks, and platform strategy that should drive the video's script
- **paid-ads** — targeting, budgets, and creative testing for the rendered ad
- **launch-strategy** — where a teaser/announcement video fits in a launch
- **product-marketing-context** — source of brand voice, colors, and positioning
