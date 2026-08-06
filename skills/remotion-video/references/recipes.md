# Remotion Marketing Recipes

Copy-paste starting points for the most common marketing videos. All examples assume Remotion 4.x, 30fps, and TypeScript. Swap colors, fonts, and copy for the brand's tokens.

## 1. Social Ad: Hook → Benefits → CTA (9:16)

A three-scene vertical ad built with `<Series>` so scenes chain automatically.

```tsx
import { AbsoluteFill, Series, interpolate, spring, useCurrentFrame, useVideoConfig, Img, staticFile } from "remotion";

const BRAND = { bg: "#0B0B0F", accent: "#5B8CFF", text: "#FFFFFF" };

const Center: React.FC<{ children: React.ReactNode }> = ({ children }) => (
  <AbsoluteFill style={{ backgroundColor: BRAND.bg, justifyContent: "center", alignItems: "center", padding: 80, textAlign: "center" }}>
    {children}
  </AbsoluteFill>
);

const RiseIn: React.FC<{ children: React.ReactNode; delay?: number }> = ({ children, delay = 0 }) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();
  const s = spring({ frame: frame - delay, fps, config: { damping: 200 } });
  return <div style={{ opacity: s, transform: `translateY(${(1 - s) * 40}px)` }}>{children}</div>;
};

export const HookBenefitsCTA: React.FC<{ hook: string; benefits: string[]; cta: string }> = ({ hook, benefits, cta }) => (
  <Series>
    <Series.Sequence durationInFrames={60}>
      <Center><RiseIn><h1 style={{ color: BRAND.text, fontSize: 100, fontWeight: 800 }}>{hook}</h1></RiseIn></Center>
    </Series.Sequence>
    <Series.Sequence durationInFrames={120}>
      <Center>
        <div style={{ display: "flex", flexDirection: "column", gap: 32 }}>
          {benefits.map((b, i) => (
            <RiseIn key={b} delay={i * 8}>
              <div style={{ color: BRAND.text, fontSize: 56 }}>✓ {b}</div>
            </RiseIn>
          ))}
        </div>
      </Center>
    </Series.Sequence>
    <Series.Sequence durationInFrames={60}>
      <Center>
        <RiseIn>
          <div style={{ backgroundColor: BRAND.accent, color: "#fff", fontSize: 64, fontWeight: 700, padding: "28px 56px", borderRadius: 20 }}>{cta}</div>
        </RiseIn>
      </Center>
    </Series.Sequence>
  </Series>
);
```

Duration = 60 + 120 + 60 = 240 frames = 8s. Register with those numbers in `Root.tsx`.

## 2. Stat / Metric Reveal (count-up)

Data-driven number that animates from 0 to its value — great for "10,000+ customers" or "3.2× ROI".

```tsx
import { AbsoluteFill, interpolate, useCurrentFrame } from "remotion";

export const StatReveal: React.FC<{ value: number; label: string; prefix?: string; suffix?: string }> = ({
  value, label, prefix = "", suffix = "",
}) => {
  const frame = useCurrentFrame();
  const progress = interpolate(frame, [0, 45], [0, 1], { extrapolateLeft: "clamp", extrapolateRight: "clamp" });
  const current = Math.round(value * progress);
  return (
    <AbsoluteFill style={{ backgroundColor: "#0B0B0F", justifyContent: "center", alignItems: "center", flexDirection: "column" }}>
      <div style={{ color: "#5B8CFF", fontSize: 200, fontWeight: 900, fontVariantNumeric: "tabular-nums" }}>
        {prefix}{current.toLocaleString()}{suffix}
      </div>
      <div style={{ color: "#fff", fontSize: 48, opacity: interpolate(frame, [20, 40], [0, 1], { extrapolateRight: "clamp" }) }}>{label}</div>
    </AbsoluteFill>
  );
};
```

`tabular-nums` keeps digits from jittering as the count changes width.

## 3. Product Screenshot Demo (Ken Burns + callout)

Slow zoom on a screenshot with an animated callout pill.

```tsx
import { AbsoluteFill, Img, interpolate, spring, staticFile, useCurrentFrame, useVideoConfig } from "remotion";

export const ScreenshotDemo: React.FC<{ src: string; note: string }> = ({ src, note }) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();
  const scale = interpolate(frame, [0, 150], [1, 1.12]); // slow Ken Burns
  const pop = spring({ frame: frame - 20, fps, config: { damping: 200 } });
  return (
    <AbsoluteFill style={{ backgroundColor: "#0B0B0F" }}>
      <AbsoluteFill style={{ transform: `scale(${scale})` }}>
        <Img src={src.startsWith("http") ? src : staticFile(src)} style={{ width: "100%", height: "100%", objectFit: "cover" }} />
      </AbsoluteFill>
      <AbsoluteFill style={{ justifyContent: "flex-end", alignItems: "center", paddingBottom: 120 }}>
        <div style={{ transform: `scale(${pop})`, backgroundColor: "#fff", color: "#0B0B0F", fontSize: 44, fontWeight: 700, padding: "20px 40px", borderRadius: 999 }}>{note}</div>
      </AbsoluteFill>
    </AbsoluteFill>
  );
};
```

## 4. Burned-in Captions (muted-first)

Show caption chunks synced to the timeline. Feed it timed segments (from a transcript or your script).

```tsx
import { AbsoluteFill, useCurrentFrame } from "remotion";

type Caption = { text: string; from: number; to: number }; // frames

export const Captions: React.FC<{ captions: Caption[] }> = ({ captions }) => {
  const frame = useCurrentFrame();
  const active = captions.find((c) => frame >= c.from && frame < c.to);
  if (!active) return null;
  return (
    <AbsoluteFill style={{ justifyContent: "flex-end", alignItems: "center", paddingBottom: 320 }}>
      <div style={{ maxWidth: "80%", textAlign: "center", color: "#fff", fontSize: 52, fontWeight: 700, lineHeight: 1.2, textShadow: "0 2px 12px rgba(0,0,0,0.9)" }}>
        {active.text}
      </div>
    </AbsoluteFill>
  );
};
```

Keep captions clear of the bottom ~15% (platform UI) and center them for cross-platform safety.

## 5. Batch / Personalized Rendering (many variants from data)

Render one composition many times with different props — one clip per plan, product, or customer. Node script using `@remotion/renderer`:

```ts
import { bundle } from "@remotion/bundler";
import { renderMedia, selectComposition } from "@remotion/renderer";
import path from "path";

const variants = [
  { headline: "Go Pro", cta: "Upgrade", out: "pro" },
  { headline: "For Teams", cta: "Book a demo", out: "teams" },
];

const serveUrl = await bundle({ entryPoint: path.resolve("src/index.ts") });

for (const v of variants) {
  const composition = await selectComposition({ serveUrl, id: "AdReel", inputProps: v });
  await renderMedia({
    composition,
    serveUrl,
    codec: "h264",
    outputLocation: `out/ad-${v.out}.mp4`,
    inputProps: v,
  });
  console.log(`Rendered ${v.out}`);
}
```

Swap the hardcoded `variants` array for rows read from a CSV/JSON or an API. This is the core pattern behind programmatic-SEO-style video: one template, hundreds of on-brand renders.

## 6. Multi-Aspect Export from One Source

Register the same component under multiple `<Composition>` ids with different `width`/`height`, then render each. Design layouts with `<AbsoluteFill>` + fl: box centering so they reflow gracefully across 9:16, 1:1, and 16:9. Use `useVideoConfig()` to branch font sizes or padding when a layout needs per-aspect tuning.

```tsx
const { width, height } = useVideoConfig();
const isVertical = height > width;
const titleSize = isVertical ? 100 : 72;
```
