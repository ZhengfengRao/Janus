# Global Entry Input Polish Implementation Plan

> **Superseded / 已过期：** This is a historical implementation plan, preserved only as execution evidence. It has been superseded by the current Janus prototype and current product design documents. Do not treat stale terminology or instructions in this file as authoritative, especially references to `Global Entry`, `Action Panel`, old entry-governance affordances, or pre-unification relationships among Janus Entry, Janus Surface, and Delegation Center.

> **For agentic workers:** REQUIRED: Use superpowers:subagent-driven-development (if subagents available) or superpowers:executing-plans to implement this plan. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Repair the Janus Global Entry input area so it matches the approved Option A: a single task capsule with Janus mark, task input, press-and-hold voice control, and submit button.

**Architecture:** This is a focused prototype edit in the existing HTML/CSS/JS file. Keep Global Entry responsible only for current task capture; remove same-row panel/governance affordances from the input capsule. Implement voice as press-and-hold interaction with clear visual/text feedback.

**Tech Stack:** Static HTML prototype with inline CSS and vanilla JavaScript.

---

## File Structure

- Modify: `.superpowers/brainstorm/73714-1779429104/janus-layout-preview.html`
  - CSS: update `.command-capsule`, `.entry-actions`, `.icon-button`, add press-and-hold voice styles, remove double-border feeling.
  - HTML: remove `打开面板` and `输入任务` chip from entry row; change voice label to `按住说话`; keep submit button.
  - JS: remove `entryPill` dependency from entry state; replace click-toggle voice with `pointerdown/pointerup/pointercancel/pointerleave` press-and-hold behavior.

## Chunk 1: Fix Global Entry Input Capsule

### Task 1: Simplify entry capsule structure

**Files:**
- Modify: `.superpowers/brainstorm/73714-1779429104/janus-layout-preview.html:207-213`
- Modify: `.superpowers/brainstorm/73714-1779429104/janus-layout-preview.html:283-289`
- Modify: `.superpowers/brainstorm/73714-1779429104/janus-layout-preview.html:345-479`

- [ ] **Step 1: Read the current prototype file**

Use Read on:

```text
/Users/rao/Desktop/private/Janus/.superpowers/brainstorm/73714-1779429104/janus-layout-preview.html
```

Expected: file contains `.command-capsule`, `entry-pill`, `open-panel`, `voice-button`, and `start-entry`.

- [ ] **Step 2: Replace entry capsule CSS**

Replace the current `.command-capsule`, `.entry-prompt`, `.entry-actions`, `.icon-button`, `.entry-close`, and `.open-panel-link` block with:

```css
  .command-capsule { display: grid; grid-template-columns: auto 1fr auto auto; align-items: center; gap: 12px; min-height: 64px; border-radius: 999px; padding: 10px 12px; background: rgba(255,255,255,.075); box-shadow: 0 22px 60px rgba(0,0,0,.34), inset 0 1px 0 rgba(255,255,255,.08); }
  .entry-prompt { color: #dbeafe; font-size: 15px; line-height: 1.45; outline: none; cursor: text; white-space: nowrap; overflow: hidden; text-overflow: ellipsis; }
  .entry-actions { display: flex; align-items: center; gap: 8px; }
  .icon-button { height: 34px; border-radius: 999px; border: 0; background: rgba(255,255,255,.1); color: #bfdbfe; font-weight: 760; cursor: pointer; padding: 0 13px; }
  .icon-button.voice-active { background: rgba(251, 113, 133, .2); color: #fecdd3; box-shadow: inset 0 0 0 1px rgba(251, 113, 133, .35); }
  .icon-button.primary { width: 34px; padding: 0; color: white; background: linear-gradient(135deg, #3b82f6, #1d4ed8); }
  .entry-close { position: absolute; right: 10px; top: 8px; width: 24px; height: 24px; border: 0; border-radius: 999px; background: rgba(15, 23, 42, .7); color: #94a3b8; cursor: pointer; }
```

Acceptance criteria:
- No `border` on `.command-capsule`.
- No `.open-panel-link` style remains necessary.
- The capsule reads visually as one surface, not a framed input inside a framed input.

- [ ] **Step 3: Replace entry HTML**

Replace the current entry actions row:

```html
<div class="entry-actions"><button class="open-panel-link" id="open-panel">打开面板</button><span class="pill" id="entry-pill">输入任务</span><button class="icon-button" id="voice-button" title="语音">◉</button><button class="icon-button primary" id="start-entry" title="开始">↵</button></div>
```

with:

```html
<div class="entry-actions"><button class="icon-button" id="voice-button" title="按住说话">按住说话</button><button class="icon-button primary" id="start-entry" title="开始">↵</button></div>
```

Acceptance criteria:
- `打开面板` is not present in the entry capsule.
- `输入任务` chip is not present in the entry capsule.
- Voice button explicitly says `按住说话`.
- Submit button remains available as `↵`.

- [ ] **Step 4: Update entry JavaScript variables**

Remove these constants:

```js
const entryPill = document.getElementById('entry-pill');
const openPanel = document.getElementById('open-panel');
```

Remove this event listener:

```js
openPanel.addEventListener('click', (event) => { event.stopPropagation(); setSurface('managed'); });
```

Acceptance criteria:
- No `entryPill` references remain.
- No `openPanel` references remain.
- Managed center remains reachable from task/executing states, not from the initial input row.

- [ ] **Step 5: Update entry state functions**

Replace:

```js
function setEntry() {
  setSurface('entry');
  entryPill.textContent = '输入任务';
  intentInput.textContent = '你想让 Janus 做什么？';
  voiceButton.textContent = '◉';
  startEntry.textContent = '↵';
}

function setIntentReady() {
  entryPill.textContent = '待开始';
  intentInput.textContent = '打开微信给张三写条消息，说我十分钟后到，发送前确认';
  voiceButton.textContent = '◉';
  startEntry.textContent = '↵';
}

function setListening() {
  entryPill.textContent = '正在听';
  intentInput.textContent = '正在听你说话，再点一次结束。';
  voiceButton.textContent = '■';
  startEntry.textContent = '等待';
}
```

with:

```js
function setEntry() {
  setSurface('entry');
  intentInput.textContent = '你想让 Janus 做什么？';
  voiceButton.textContent = '按住说话';
  voiceButton.classList.remove('voice-active');
  startEntry.textContent = '↵';
}

function setIntentReady() {
  intentInput.textContent = '打开微信给张三写条消息，说我十分钟后到，发送前确认';
  voiceButton.textContent = '按住说话';
  voiceButton.classList.remove('voice-active');
  startEntry.textContent = '↵';
}

function startListening() {
  intentInput.textContent = '正在听你说话，松开结束。';
  voiceButton.textContent = '松开结束';
  voiceButton.classList.add('voice-active');
  startEntry.textContent = '等待';
}

function stopListening() {
  setIntentReady();
}
```

Acceptance criteria:
- The interaction copy says `正在听你说话，松开结束。` while pressing.
- There is no “再点一次结束” copy.
- Voice active state is visual, temporary, and reversible.

- [ ] **Step 6: Replace voice event handling**

Replace the current voice click listener:

```js
voiceButton.addEventListener('click', (event) => {
  event.stopPropagation();
  if (voiceButton.textContent === '结束') setIntentReady();
  else setListening();
});
```

with:

```js
voiceButton.addEventListener('pointerdown', (event) => {
  event.stopPropagation();
  startListening();
});
voiceButton.addEventListener('pointerup', (event) => {
  event.stopPropagation();
  stopListening();
});
voiceButton.addEventListener('pointercancel', stopListening);
voiceButton.addEventListener('pointerleave', () => {
  if (voiceButton.classList.contains('voice-active')) stopListening();
});
```

Acceptance criteria:
- Pressing starts listening.
- Releasing ends listening and fills the demo task.
- Moving/canceling does not leave the UI stuck in listening state.

- [ ] **Step 7: Verify with static text search**

Use Grep for these patterns in the prototype file:

```text
打开面板
输入任务
再点一次结束
entryPill
openPanel
```

Expected:
- `打开面板` may remain elsewhere only if unrelated to Global Entry copy; it must not appear inside entry capsule HTML.
- `输入任务`, `再点一次结束`, `entryPill`, and `openPanel` should have no matches.

- [ ] **Step 8: Open/preview the visual companion page**

Use the existing visual server URL if still running:

```text
http://localhost:51546
```

Or open the modified HTML file directly in a browser.

Acceptance criteria:
- Initial entry capsule has exactly four elements: Janus mark, input text, `按住说话`, submit.
- No outer input border inside the Janus surface.
- No same-row panel/governance affordance.
- Voice behaves as press-and-hold.

- [ ] **Step 9: Report completion with evidence**

Report:
- modified file path;
- removed elements;
- voice interaction behavior;
- remaining known risk, if any.

Do not claim completion unless the static checks pass and the visual state has been inspected.
