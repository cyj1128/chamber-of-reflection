# Cinematic Transition Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Strengthen the continuity between sections with restrained cinematic morph transitions, longer Echo/Reflection cursor behavior, and soft parallax without breaking the current mood or media-driven scene structure.

**Architecture:** Keep the single-file `index.html` architecture, but expand the existing scroll state model to compute boundary transition progress between adjacent sections. Use that boundary progress inside the current canvas overlay renderer so scenes visually transform into the next scene instead of simply swapping. Keep media layers as the base scene and use canvas/HUD timing for subtle transition behavior.

**Tech Stack:** Vanilla HTML/CSS/JavaScript, canvas 2D overlay renderer, fixed media layers, Git

---

### Task 1: Add Boundary Transition State

**Files:**
- Modify: `/Users/seomahyeog/Desktop/vibecoding/ddd/4/index.html`

- [ ] **Step 1: Add transition state variables near the existing scroll state**

Add these variables near the existing `let sectionProgress = 0;` declarations:

```js
      let sectionProgress = 0;
      let currentSceneIndex = 0;
      let globalProgress = 0;
      let boundaryProgress = 0;
      let boundaryFromKey = "intro";
      let boundaryToKey = "silence";
```

- [ ] **Step 2: Add a helper that computes cross-section boundary progress**

Add this helper below `easeOutCubic`:

```js
      function mapBoundaryProgress(progress) {
        if (progress <= 0.8) {
          return 0;
        }

        return clamp((progress - 0.8) / 0.2, 0, 1);
      }
```

- [ ] **Step 3: Update scroll state to calculate from/to section keys**

Inside `updateScrollState()`, immediately after:

```js
        currentSceneIndex = activeIndex;
        sectionProgress = activeProgress;
```

replace the single-state handling with:

```js
        currentSceneIndex = activeIndex;
        sectionProgress = activeProgress;

        const currentMeta = sceneMeta[currentSceneIndex];
        const nextMeta = sceneMeta[Math.min(currentSceneIndex + 1, sceneMeta.length - 1)];

        boundaryProgress = mapBoundaryProgress(sectionProgress);
        boundaryFromKey = currentMeta.key;
        boundaryToKey = nextMeta.key;
```

- [ ] **Step 4: Update the existing meta usage to use `currentMeta`**

Replace:

```js
        const meta = sceneMeta[currentSceneIndex];
        sectionCount.textContent = meta.number;
        sectionName.textContent = meta.name;
        document.body.classList.toggle("is-intro", meta.key === "intro");
        document.body.classList.toggle("is-silence", meta.key === "silence");
```

with:

```js
        sectionCount.textContent = currentMeta.number;
        sectionName.textContent = currentMeta.name;
        document.body.classList.toggle("is-intro", currentMeta.key === "intro");
        document.body.classList.toggle("is-silence", currentMeta.key === "silence");
```

- [ ] **Step 5: Verify the file still parses cleanly**

Run:

```bash
node -e "const fs=require('fs'); const html=fs.readFileSync('index.html','utf8'); const js=html.split('<script>')[1].split('</script>')[0]; new Function(js); console.log('ok');"
```

Expected:

```text
ok
```

- [ ] **Step 6: Commit**

```bash
git add /Users/seomahyeog/Desktop/vibecoding/ddd/4/index.html
git commit -m "feat: add boundary transition state"
```

### Task 2: Add Transition Overlay Renderers

**Files:**
- Modify: `/Users/seomahyeog/Desktop/vibecoding/ddd/4/index.html`

- [ ] **Step 1: Add a glow-point helper**

Add this helper below `drawHorizon`:

```js
      function drawGlowPoint(x, y, radius, alpha) {
        const gradient = ctx.createRadialGradient(x, y, 0, x, y, radius);
        gradient.addColorStop(0, `rgba(255,255,255,${alpha})`);
        gradient.addColorStop(0.25, `rgba(255,255,255,${alpha * 0.35})`);
        gradient.addColorStop(1, "rgba(255,255,255,0)");
        ctx.fillStyle = gradient;
        ctx.beginPath();
        ctx.arc(x, y, radius, 0, Math.PI * 2);
        ctx.fill();
      }
```

- [ ] **Step 2: Add a helper for drawing repeated loop seeds**

Add this helper below `drawGlowPoint`:

```js
      function drawLoopSeeds(progress) {
        const count = Math.floor(lerp(2, 14, progress));
        const gap = lerp(18, 54, progress);
        const centerY = height * 0.5;
        const centerX = width * 0.5;

        for (let i = 0; i < count; i += 1) {
          const offset = gap * i;
          const size = lerp(3.5, 10, i / Math.max(count - 1, 1));
          const alpha = lerp(0.07, 0.24, progress) * (1 - i / (count + 2));
          drawGlowPoint(centerX - offset, centerY, size, alpha);
          drawGlowPoint(centerX + offset, centerY, size, alpha);
        }
      }
```

- [ ] **Step 3: Add a helper for drawing Echo trails**

Add this helper below `drawLoopSeeds`:

```js
      function drawEchoRibbon(progress) {
        const centerY = height * 0.5;
        const startX = width * 0.34;
        const endX = width * 0.66;
        const steps = 22;

        for (let i = 0; i < steps; i += 1) {
          const t = i / (steps - 1);
          const x = lerp(startX, endX, t);
          const radius = lerp(8, 34, progress) * (1 - t * 0.6);
          const alpha = lerp(0.04, 0.14, progress) * (1 - t * 0.55);
          drawGlowPoint(x, centerY, radius, alpha);
        }
      }
```

- [ ] **Step 4: Add a transition renderer**

Add this function below the new helpers:

```js
      function renderTransitionOverlay(fromKey, toKey, progress) {
        if (progress <= 0) {
          return;
        }

        const eased = easeOutCubic(progress);
        const centerX = width * 0.5;
        const centerY = height * 0.5;

        if (fromKey === "silence" && toKey === "loop") {
          drawGlowPoint(centerX, centerY, lerp(48, 22, eased), lerp(0.18, 0.08, eased));
          drawLoopSeeds(eased);
          return;
        }

        if (fromKey === "loop" && toKey === "echo") {
          drawLoopSeeds(1 - eased * 0.45);
          drawEchoRibbon(eased);
          return;
        }

        if (fromKey === "echo" && toKey === "reflection") {
          drawEchoRibbon(1 - eased * 0.2);
          drawHorizon(centerY, lerp(0.02, 0.12, eased));
          drawGlowPoint(centerX, centerY, lerp(24, 48, eased), lerp(0.08, 0.14, eased));
          return;
        }

        if (fromKey === "reflection" && toKey === "isolation") {
          drawHorizon(centerY, lerp(0.1, 0.02, eased));
          drawGlowPoint(centerX, centerY, lerp(42, 22, eased), lerp(0.14, 0.1, eased));
          return;
        }

        if (fromKey === "isolation" && toKey === "disappearance") {
          drawGlowPoint(centerX, centerY, lerp(20, 10, eased), lerp(0.12, 0.06, eased));
        }
      }
```

- [ ] **Step 5: Call the transition renderer inside the main animation tick**

In `tick()`, after the existing `renderSceneOverlay(...)` switch block, add:

```js
        renderTransitionOverlay(boundaryFromKey, boundaryToKey, boundaryProgress);
```

- [ ] **Step 6: Verify the file still parses cleanly**

Run:

```bash
node -e "const fs=require('fs'); const html=fs.readFileSync('index.html','utf8'); const js=html.split('<script>')[1].split('</script>')[0]; new Function(js); console.log('ok');"
```

Expected:

```text
ok
```

- [ ] **Step 7: Commit**

```bash
git add /Users/seomahyeog/Desktop/vibecoding/ddd/4/index.html
git commit -m "feat: add cinematic transition overlays"
```

### Task 3: Strengthen Echo / Reflection / Disappearance Interaction

**Files:**
- Modify: `/Users/seomahyeog/Desktop/vibecoding/ddd/4/index.html`

- [ ] **Step 1: Increase trail retention for Echo**

Find `updateTrails()` and change the trail decay block from:

```js
          point.life *= 0.94;
```

to:

```js
          const sceneKey = sceneMeta[currentSceneIndex].key;
          const decay =
            sceneKey === "echo" ? 0.972 :
            sceneKey === "reflection" ? 0.965 :
            0.94;
          point.life *= decay;
```

- [ ] **Step 2: Increase retained buffer lengths**

In `addTrailPoint`, change:

```js
        if (buffer.length > 48) {
```

to:

```js
        const sceneKey = sceneMeta[currentSceneIndex].key;
        const maxLength = sceneKey === "echo" ? 88 : sceneKey === "reflection" ? 72 : 48;
        if (buffer.length > maxLength) {
```

- [ ] **Step 3: Add scene-weighted delayed trail amplification**

Inside `renderSceneOverlay`, in the `reflection` case, ensure the delayed layers are more visible by adding:

```js
          delayedTrail.forEach((point, index) => {
            const fade = point.life * (1 - index / delayedTrail.length);
            drawGlowPoint(point.x, point.y, 10 + point.radius * 5, fade * 0.045);
          });

          delayedTrailFar.forEach((point, index) => {
            const fade = point.life * (1 - index / delayedTrailFar.length);
            drawGlowPoint(point.x, point.y, 16 + point.radius * 6, fade * 0.03);
          });
```

Use this only if those arrays are not already rendered with equivalent visibility in the reflection branch. Replace the weaker version instead of duplicating it.

- [ ] **Step 4: Add subtle cursor disturbance in Disappearance**

In the `disappearance` branch of `renderSceneOverlay`, after the particle position is computed, adjust it with pointer influence:

```js
            const dx = x - pointer.x;
            const dy = y - pointer.y;
            const distance = Math.hypot(dx, dy) || 1;
            const push = Math.max(0, 1 - distance / 140) * 8;
            x += (dx / distance) * push;
            y += (dy / distance) * push;
```

Apply this with local `let x` and `let y` variables so the position can be offset.

- [ ] **Step 5: Verify the file still parses cleanly**

Run:

```bash
node -e "const fs=require('fs'); const html=fs.readFileSync('index.html','utf8'); const js=html.split('<script>')[1].split('</script>')[0]; new Function(js); console.log('ok');"
```

Expected:

```text
ok
```

- [ ] **Step 6: Commit**

```bash
git add /Users/seomahyeog/Desktop/vibecoding/ddd/4/index.html
git commit -m "feat: enhance echo and reflection interactions"
```

### Task 4: Add Soft Parallax Without Shaking Media

**Files:**
- Modify: `/Users/seomahyeog/Desktop/vibecoding/ddd/4/index.html`

- [ ] **Step 1: Add a transition for section content movement**

In the `.section__content` CSS block, add:

```css
        will-change: transform, opacity;
        transition: transform 220ms ease, opacity 220ms ease;
```

- [ ] **Step 2: Add a helper that updates text parallax**

Add this function below `updateSceneImages()`:

```js
      function updateTextParallax() {
        sections.forEach((section, index) => {
          const content = section.querySelector(".section__content");
          if (!content) return;

          const rect = section.getBoundingClientRect();
          const local = clamp((height * 0.5 - rect.top) / (rect.height + height * 0.5), 0, 1);
          const shiftY = lerp(12, -12, local);
          const opacity = index === currentSceneIndex ? 1 : 0.92;
          content.style.transform = `translate3d(0, ${shiftY}px, 0)`;
          content.style.opacity = String(opacity);
        });
      }
```

- [ ] **Step 3: Use the helper in the animation loop**

Inside `tick()`, after `updateSceneImages();`, add:

```js
        updateTextParallax();
```

- [ ] **Step 4: Reduce media movement for all video-based scenes**

In `updateSceneImages()`, keep `loop` fixed as it is now, and also make `silence` and `echo` more restrained by updating these branches to:

```js
          if (key === "silence") {
            scale = 1.01 + sectionProgress * 0.01;
            translateX = px * 4;
            translateY = py * 4;
            opacity = active ? 0.34 : 0;
          } else if (key === "loop") {
            scale = 1.02;
            translateX = 0;
            translateY = 0;
            rotate = 0;
            opacity = active ? 0.72 : 0;
          } else if (key === "echo") {
            scale = 1.02 + sectionProgress * 0.03;
            translateX = px * 6;
            translateY = py * 6;
            rotate = px * 0.3;
            opacity = active ? 0.62 : 0;
          }
```

- [ ] **Step 5: Verify the file still parses cleanly**

Run:

```bash
node -e "const fs=require('fs'); const html=fs.readFileSync('index.html','utf8'); const js=html.split('<script>')[1].split('</script>')[0]; new Function(js); console.log('ok');"
```

Expected:

```text
ok
```

- [ ] **Step 6: Commit**

```bash
git add /Users/seomahyeog/Desktop/vibecoding/ddd/4/index.html
git commit -m "feat: add restrained text and scene parallax"
```

### Task 5: Verify, Review, And Publish

**Files:**
- Modify: `/Users/seomahyeog/Desktop/vibecoding/ddd/4/index.html`
- Check: `/Users/seomahyeog/Desktop/vibecoding/ddd/4/docs/superpowers/specs/2026-06-09-cinematic-transition-design.md`

- [ ] **Step 1: Start a local preview server**

Run:

```bash
python3 -m http.server 4173
```

Expected:

```text
Serving HTTP on 0.0.0.0 port 4173 ...
```

- [ ] **Step 2: Confirm the page responds**

Run:

```bash
curl -I http://127.0.0.1:4173/index.html
```

Expected:

```text
HTTP/1.0 200 OK
```

- [ ] **Step 3: Manual browser QA checklist**

Verify these behaviors manually in the browser:

```text
1. Intro title still fades in and auto-transitions into Silence.
2. Silence -> Loop now feels like one point becoming repeated seeds.
3. Loop -> Echo feels softer and more smeared, not a hard swap.
4. Echo cursor trails last longer and visibly linger.
5. Reflection cursor trails feel delayed and layered.
6. Isolation still feels the calmest and least reactive.
7. Disappearance particles react subtly near the cursor.
8. Videos in Silence / Loop / Echo still render and do not jump unexpectedly.
```

- [ ] **Step 4: Commit final implementation polish if needed**

```bash
git add /Users/seomahyeog/Desktop/vibecoding/ddd/4/index.html
git commit -m "fix: polish cinematic transition behavior"
```

Only create this commit if QA required actual code changes.

- [ ] **Step 5: Push the updated branch**

```bash
git push origin main
```

Expected:

```text
To https://github.com/cyj1128/chamber-of-reflection.git
   <old>..<new>  main -> main
```
