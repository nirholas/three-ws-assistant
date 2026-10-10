# @three-ws/assistant

<!-- three.ws:badges -->
[![GitHub stars](https://img.shields.io/github/stars/nirholas/three-ws-assistant?style=flat&logo=github)](https://github.com/nirholas/three-ws-assistant/stargazers) [![License](https://img.shields.io/github/license/nirholas/three-ws-assistant?style=flat)](https://github.com/nirholas/three-ws-assistant/blob/HEAD/LICENSE) [![Last commit](https://img.shields.io/github/last-commit/nirholas/three-ws-assistant?style=flat)](https://github.com/nirholas/three-ws-assistant/commits) [![PRs welcome](https://img.shields.io/badge/PRs-welcome-brightgreen?style=flat)](https://github.com/nirholas/three-ws-assistant/pulls) [![AI agent friendly](https://img.shields.io/badge/AI%20agents-AGENTS.md%20%2B%20llms.txt-6d5dfc?style=flat)](https://github.com/nirholas/three-ws-assistant/blob/HEAD/AGENTS.md)
<!-- /three.ws:badges -->


A 3D avatar assistant for any website, in one script tag. A floating launcher opens a real, animated 3D avatar in a panel, standing directly on your page (transparent), or against a color or gradient. It has two modes:

- **Chat**: a real chatbot. Replies stream into a speech bubble above the avatar's head and optionally speak aloud. Default answers come from the platform's free model chain (no key, no account). Visitors can switch to their own Groq or OpenRouter key in the widget settings; the key stays in their browser.
- **Speak**: the avatar says exactly what you type, out loud, with a speech bubble. No model involved, browser text-to-speech, zero cost.

The avatar, chat, and speech run inside an iframe hosted on three.ws (`/assistant-frame`), so your page never sees model keys and the frame never sees your DOM. This package is the small host-side loader.

- Live builder and demo: [three.ws/assistant](https://three.ws/assistant)
- Docs: [three.ws/docs/assistant-widget](https://three.ws/docs/assistant-widget)
- Generate an MCP-driven config from any agent: [`@three-ws/assistant-mcp`](https://www.npmjs.com/package/@three-ws/assistant-mcp)

## Install

### 1. One tag (no build)

```html
<script src="https://three.ws/assistant/v1.js" async
  data-avatar="/avatars/selfie-girl.glb"
  data-name="Atelier AI"
  data-greeting="Ask anything about Atelier."
  data-context="Atelier is a design studio for 3D artists."
  data-bg="transparent"
  data-accent="#f97316"></script>
```

### 2. npm

```bash
npm install @three-ws/assistant
```

```js
import ThreeAssistant from '@three-ws/assistant';

ThreeAssistant.init({
  avatar: '/avatars/selfie-girl.glb',
  name: 'Atelier AI',
  greeting: 'Ask anything about Atelier.',
  context: 'Atelier is a design studio for 3D artists.',
  bg: 'transparent',
});
```

## Options

Every option is a config key (npm) or a `data-*` attribute (one-tag). Unknown or invalid values are re-validated inside the frame and fall back to a safe default, so a bad value never breaks the page.

| Key / attribute | Values | Default | Description |
| --- | --- | --- | --- |
| `avatar` / `data-avatar` | avatar id, `/avatars/*.glb`, or a GLB URL | default mannequin | Which body to load. External URLs must be GLB/GLTF/VRM on an allow-listed host (three.ws, R2, Ready Player Me). |
| `agent` / `data-agent` | a three.ws agent id | none | Load that agent's avatar instead. |
| `bg` / `data-bg` | `transparent`, `#hex`, a preset (`ember`, `ocean`, `violet`, `forest`, `dusk`, `slate`), or `gradient:#a,#b,angle` | `transparent` | `transparent` floats the avatar over your page; anything else paints the panel. |
| `mode` / `data-mode` | `chat`, `speak`, `both` | `both` | `both` shows a Chat / Speak toggle. |
| `name` / `data-name` | text (60) | `Assistant` | Header title and chatbot persona name. |
| `greeting` / `data-greeting` | text (200) | mode default | First speech bubble on open. |
| `context` / `data-context` | text (500) | none | What the chatbot should know about your site. |
| `accent` / `data-accent` | `#hex` | `#f97316` | Launcher, send button, focus rings. |
| `position` / `data-position` | `right`, `left` | `right` | Launcher corner. |
| `voice` / `data-voice` | `false` | on | Start with voice muted. |
| `badge` / `data-badge` | `false` | on | Hide the three.ws attribution badge. |
| `open` / `data-open` | present | closed | Start with the panel open. |
| `origin` | URL | `https://three.ws` | Frame host (self-hosting only; npm config key). |
| `data-manual` | present | auto | One-tag only: do not auto-mount; call `ThreeAssistant.init()` yourself. |

## API

`window.ThreeAssistant` (one-tag) and the default export (npm) share the same API:

```js
ThreeAssistant.init(config);   // replaces any existing instance, returns the Assistant
ThreeAssistant.open();
ThreeAssistant.close();
ThreeAssistant.toggle();
ThreeAssistant.say('Welcome to the sale!'); // opens and speaks the line aloud
ThreeAssistant.setMode('speak');            // or 'chat' (only when mounted with mode 'both')
ThreeAssistant.destroy();
ThreeAssistant.instance;        // the live Assistant, or null
```

Named exports are also available for tree-shaking bundlers:

```js
import { init, say, createAssistant, VERSION } from '@three-ws/assistant';
```

`createAssistant({ origin })` returns an isolated API bound to a specific frame host, if you run more than one or self-host.

## Events

Listen once on `window` for the widget lifecycle:

```js
window.addEventListener('three-assistant', (e) => {
  // e.detail.type: 'ready' | 'open' | 'close' | 'message' | 'speak:start' | 'speak:end' | 'error'
  if (e.detail.type === 'message') {
    console.log(e.detail.payload.role, e.detail.payload.content);
  }
});
```

## How it works

The launcher and panel are plain DOM this package injects. The panel holds an `<iframe>` pointing at `https://three.ws/assistant-frame?<your config>`. Inside that frame: a Three.js avatar with idle, wave, and talking animations; a head-anchored speech bubble; streaming chat over the platform's free model chain (or the visitor's own key, sent browser-to-provider); and Web Speech text-to-speech. Config crosses to the frame as query params and is re-validated there, so a hostile embedding page cannot inject CSS or URLs. Host and frame talk over a tiny `postMessage` protocol on the `three-assistant` channel.

## Develop

```bash
npm install     # esbuild for the build, jsdom for the DOM tests
npm run build   # esbuild -> dist/assistant.mjs + dist/assistant.global.js
npm test        # node --test: the pure loader logic, plus the mounted widget under jsdom
npx serve .     # after build, open http://localhost:3000/examples/
```

Serve this package root, not `examples/`: the example page loads the build you
just made from `../dist/`, which a server rooted at `examples/` cannot reach.
The page talks to the live `three.ws` frame, so chat, voice, and the avatar all
work from `localhost` with no extra setup.

## License

Proprietary. Free to embed on any site via the published builds (the one-tag CDN script or this npm package). See [LICENSE](./LICENSE).

<!-- three.ws:growth -->
## Support the project

If three-ws-assistant saves you time, **[star it on GitHub](https://github.com/nirholas/three-ws-assistant)**. Stars are how other developers and AI agents find the repositories worth trusting, and they cost you one click.

Know someone who would use it? [Post on X](https://twitter.com/intent/tweet?text=three-ws-assistant%3A%20A%203D%20avatar%20assistant%20for%20any%20website%20in%20one%20script%20tag&url=https%3A%2F%2Fgithub.com%2Fnirholas%2Fthree-ws-assistant) · [Share on Bluesky](https://bsky.app/intent/compose?text=three-ws-assistant%3A%20A%203D%20avatar%20assistant%20for%20any%20website%20in%20one%20script%20tag%20https%3A%2F%2Fgithub.com%2Fnirholas%2Fthree-ws-assistant) · [Share on LinkedIn](https://www.linkedin.com/sharing/share-offsite/?url=https%3A%2F%2Fgithub.com%2Fnirholas%2Fthree-ws-assistant) · [Submit to Hacker News](https://news.ycombinator.com/submitlink?u=https%3A%2F%2Fgithub.com%2Fnirholas%2Fthree-ws-assistant&t=three-ws-assistant%3A%20A%203D%20avatar%20assistant%20for%20any%20website%20in%20one%20script%20tag) · [Share on Reddit](https://www.reddit.com/submit?url=https%3A%2F%2Fgithub.com%2Fnirholas%2Fthree-ws-assistant&title=three-ws-assistant%3A%20A%203D%20avatar%20assistant%20for%20any%20website%20in%20one%20script%20tag)

## Built for AI agents too

Coding agents and LLM tooling can read this repo directly: [AGENTS.md](./AGENTS.md), [llms.txt](./llms.txt), [llms-full.txt](./llms-full.txt). Point an agent at `https://github.com/nirholas/three-ws-assistant` and it has the context it needs.

## More from the same author

- [All repositories by nirholas](https://github.com/nirholas/nirholas#readme): the full catalog, grouped by topic
- [three.ws](https://three.ws): the platform for 3D AI agents with Solana wallets, a skill marketplace and x402 payments
- Questions or ideas: [open an issue](https://github.com/nirholas/three-ws-assistant/issues) or [start a discussion](https://github.com/nirholas/three-ws-assistant/discussions)

## Contributors

[![Contributors](https://contrib.rocks/image?repo=nirholas/three-ws-assistant)](https://github.com/nirholas/three-ws-assistant/graphs/contributors)

## Star history

[![Star History Chart](https://api.star-history.com/svg?repos=nirholas/three-ws-assistant&type=Date)](https://www.star-history.com/#nirholas/three-ws-assistant&Date)

<!-- /three.ws:growth -->

<!-- three.ws:ecosystem -->
## Part of three.ws

[three.ws](https://three.ws) is a platform for 3D AI agents with Solana wallets: avatars, a skill marketplace, x402 payments and more than seventy MCP servers. `@three-ws/assistant` is one package from it.

- Documentation site: https://nirholas.github.io/three-ws-assistant/
- npm: https://www.npmjs.com/package/@three-ws/assistant
- Canonical source: https://github.com/nirholas/three.ws (this repository is a generated mirror of `assistant-sdk`)
- Agent-readable summary: [llms.txt](./llms.txt) and [AGENTS.md](./AGENTS.md)
- Issues and ideas: https://github.com/nirholas/three-ws-assistant/issues
<!-- /three.ws:ecosystem -->
