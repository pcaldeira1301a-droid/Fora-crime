#!/usr/bin/env bash
# Cria o repositório "safe-community" no GitHub com o site e liga o GitHub Pages.
# Uso:  bash criar-site-github.sh [nome-do-repositorio]
# Precisa do GitHub CLI (gh): https://cli.github.com
set -euo pipefail

REPO="${1:-safe-community}"

if ! command -v gh >/dev/null 2>&1; then
  echo "Instale o GitHub CLI (gh) primeiro: https://cli.github.com" >&2
  exit 1
fi
if ! command -v git >/dev/null 2>&1; then
  echo "Instale o git primeiro: https://git-scm.com" >&2
  exit 1
fi

# Entra na conta do GitHub se ainda não estiver logado
gh auth status >/dev/null 2>&1 || gh auth login

LOGIN="$(gh api user --jq .login)"
DIR="$(mktemp -d)"
cd "$DIR"

cat > index.html <<'SITE_EOF'
<!doctype html>
<html lang="en">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1, viewport-fit=cover">
<title>Safe Community</title>
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link href="https://fonts.googleapis.com/css2?family=Chakra+Petch:wght@600;700&family=Figtree:wght@400;500;600;700&family=Patrick+Hand&display=swap" rel="stylesheet">
<style>
  [hidden] { display: none !important; }
  :root {
    --bg: #fbe3b4;
    --frame: #f0a63a;
    --panel: #f4faf8;
    --surface: #ffffff;
    --fg: #16262b;
    --muted: #46606a;
    --teal: #17796a;
    --mint: #cfeee3;
    --violet: #4437c9;
    --yellow: #f2b705;
    --line: #5db69d;
    --pin: #2f6fd6;
    --on-teal: #ffffff;
    --shadow: rgba(20, 38, 43, .25);
    --font-display: "Chakra Petch", "Trebuchet MS", system-ui, sans-serif;
    --font-body: "Figtree", system-ui, -apple-system, "Segoe UI", sans-serif;
    --font-hand: "Patrick Hand", "Comic Sans MS", cursive;
  }
  @media (prefers-color-scheme: dark) {
    :root:not([data-theme="light"]) {
      --bg: #1e1608;
      --frame: #c98421;
      --panel: #13262a;
      --surface: #1a3237;
      --fg: #e8f3f0;
      --muted: #a2bcb6;
      --teal: #2fae98;
      --mint: #1c4039;
      --violet: #aaa2ff;
      --yellow: #f5c542;
      --line: #3a7d6e;
      --pin: #6c9cf5;
      --on-teal: #0e1b1e;
      --shadow: rgba(0, 0, 0, .5);
      color-scheme: dark;
    }
  }
  :root[data-theme="dark"] {
    --bg: #1e1608;
    --frame: #c98421;
    --panel: #13262a;
    --surface: #1a3237;
    --fg: #e8f3f0;
    --muted: #a2bcb6;
    --teal: #2fae98;
    --mint: #1c4039;
    --violet: #aaa2ff;
    --yellow: #f5c542;
    --line: #3a7d6e;
    --pin: #6c9cf5;
    --on-teal: #0e1b1e;
    --shadow: rgba(0, 0, 0, .5);
    color-scheme: dark;
  }

  * { box-sizing: border-box; }
  body {
    margin: 0;
    background: var(--bg);
    color: var(--fg);
    font-family: var(--font-body);
    font-size: 17px;
    line-height: 1.55;
    padding-block: 24px 36px;
    padding-left: max(16px, env(safe-area-inset-left, 0px));
    padding-right: max(16px, env(safe-area-inset-right, 0px));
  }
  h1, h2, h3, p, ul, ol, dl, dd, figure { margin: 0; }
  h1, h2, h3 { font-family: var(--font-display); text-wrap: balance; line-height: 1.1; }
  a:focus-visible { outline: 3px solid var(--yellow); outline-offset: 3px; }
  svg { display: block; max-width: 100%; }

  .frame {
    max-width: 1060px; margin: 0 auto;
    background: var(--frame); border-radius: 26px; padding: 14px;
    box-shadow: 0 18px 40px -18px var(--shadow);
  }
  .poster {
    position: relative; background: var(--panel); border-radius: 16px;
    padding: 34px clamp(18px, 4vw, 44px) 30px;
    display: grid; gap: 26px;
  }
  .poster::before, .poster::after {
    content: ""; position: absolute; top: 14px; width: 14px; height: 14px; border-radius: 50%;
    background: var(--pin); box-shadow: inset -2px -2px 0 rgba(0,0,0,.25);
  }
  .poster::before { left: 14px; }
  .poster::after { right: 14px; }

  .top { display: grid; gap: 12px; justify-items: center; text-align: center; }
  .tag {
    display: inline-flex; align-items: center; gap: 10px; flex-wrap: wrap; justify-content: center;
    background: var(--teal); color: var(--on-teal); border-radius: 6px; padding: 8px 18px;
    font-family: var(--font-display); font-weight: 600; letter-spacing: .08em; text-transform: uppercase; font-size: 14px;
  }
  .tag svg { width: 20px; height: 20px; flex: none; }
  .tag span { font-weight: 400; text-transform: none; letter-spacing: .02em; opacity: .9; }
  .top h1 {
    color: var(--violet); text-transform: uppercase; letter-spacing: .01em;
    font-size: clamp(1.9rem, 6.6vw, 3.6rem);
  }
  .top .sub { font-size: clamp(1.05rem, 2.4vw, 1.3rem); color: var(--muted); max-width: 44ch; }

  .eyebrow {
    font-family: var(--font-display); font-weight: 700; text-transform: uppercase; letter-spacing: .05em;
    color: var(--violet); font-size: 1rem; margin-bottom: 14px;
  }

  .grid { display: grid; grid-template-columns: 1.1fr 1fr; gap: 34px; align-items: start; }
  .grid > * { min-width: 0; }
  @media (max-width: 820px) { .grid { grid-template-columns: 1fr; } }

  .ideas { list-style: none; padding: 0; display: grid; gap: 14px; }
  .ideas li { display: grid; grid-template-columns: 34px 1fr; gap: 12px; align-items: start; }
  .ideas .n {
    font-family: var(--font-display); font-weight: 700; font-size: 1.9rem; line-height: 1; color: var(--teal);
  }
  .ideas b { font-weight: 700; }

  .mapfig { position: relative; margin: 30px 0 70px; }
  .map { border-radius: 4px; overflow: hidden; box-shadow: 0 8px 20px -10px var(--shadow); }
  .map svg { width: 100%; height: auto; }
  .maplabel {
    position: absolute; left: 0; bottom: 0; background: #1a1f22; color: #fff;
    font-family: var(--font-display); font-weight: 600; font-size: 15px; letter-spacing: .03em;
    padding: 7px 18px;
  }
  .zoom {
    position: absolute; right: 4%; bottom: -54px; width: min(34%, 170px); aspect-ratio: 1;
    border-radius: 50%; border: 5px solid #fff; overflow: hidden; box-shadow: 0 10px 22px -8px var(--shadow);
    background: #67b05a;
  }
  .zoom svg { width: 100%; height: 100%; }
  .zoomlabel {
    position: absolute; right: calc(4% + min(34%, 170px) / 2); bottom: -76px; transform: translateX(50%);
    background: var(--surface); color: var(--fg); border: 1.5px solid var(--fg);
    font-size: 14px; font-weight: 600; padding: 1px 10px; white-space: nowrap;
  }

  .right { display: grid; gap: 22px; }
  .event {
    position: relative; background: var(--mint); padding: 16px 20px 18px; text-align: center; display: grid; gap: 4px;
    font-family: var(--font-hand); font-size: 1.3rem; line-height: 1.3;
  }
  .event .label {
    font-family: var(--font-display); font-weight: 700; text-transform: uppercase; letter-spacing: .05em;
    font-size: 1rem; line-height: 1.2;
  }
  .chip {
    justify-self: center; font-family: var(--font-body); font-weight: 600; font-size: 12px;
    background: var(--yellow); color: #1b1600; padding: 2px 10px; border-radius: 99px;
  }
  .arrow {
    position: absolute; left: -52px; top: -28px; width: 54px; height: 54px; color: var(--yellow);
  }
  @media (max-width: 820px) { .arrow { display: none; } }

  .why {
    border: 3.5px dotted var(--line); border-radius: 22px; padding: 22px; display: grid; gap: 14px;
  }
  .why h2 { font-size: clamp(1.3rem, 3.2vw, 1.7rem); }
  .why .photo { border-radius: 6px; overflow: hidden; }
  .why .photo svg { width: 100%; height: auto; }
  .why ul { list-style: none; padding: 0; display: grid; gap: 8px; }
  .why li { display: grid; grid-template-columns: 14px 1fr; gap: 8px; }
  .why li::before { content: "•"; font-weight: 700; color: var(--teal); }

  .message {
    font-family: var(--font-display); font-weight: 700; text-transform: uppercase; letter-spacing: .03em;
    color: var(--violet); font-size: 1.1rem; text-align: center; line-height: 1.3;
  }

  .closing {
    text-align: center; color: var(--violet); text-transform: uppercase; letter-spacing: .02em;
    font-size: clamp(1.4rem, 4.6vw, 2.4rem); padding-top: 4px;
  }

  .contact { text-align: center; color: var(--muted); font-size: 15px; display: grid; gap: 4px; }
  .contact strong { color: var(--fg); }

  .btn {
    appearance: none; font: inherit; font-weight: 700; cursor: pointer; text-decoration: none;
    display: inline-flex; align-items: center; justify-content: center; gap: 8px; min-height: 44px;
    border-radius: 10px; padding: 10px 18px; border: 2px solid var(--teal); background: var(--teal); color: var(--on-teal);
  }
  .btn.alt { background: transparent; color: var(--teal); }
  .btn:focus-visible { outline: 3px solid var(--yellow); outline-offset: 3px; }
  .btn[disabled], .btn[aria-disabled="true"] { opacity: .5; cursor: not-allowed; }
  .btn svg { width: 20px; height: 20px; flex: none; }
  .sr { position: absolute; width: 1px; height: 1px; overflow: hidden; clip: rect(0 0 0 0); white-space: nowrap; }

  .hintline { font-size: 15px; color: var(--muted); margin: -4px 0 14px; }
  .ideas li { display: block; }
  .idea {
    appearance: none; width: 100%; text-align: left; cursor: pointer; font: inherit; color: var(--fg);
    display: grid; grid-template-columns: 34px 1fr auto; gap: 12px; align-items: center;
    background: var(--surface); border: 2px solid var(--line); border-radius: 14px; padding: 12px 14px;
    transition: transform .15s ease, border-color .15s ease;
  }
  .idea:hover { border-color: var(--violet); transform: translateY(-1px); }
  .idea:focus-visible { outline: 3px solid var(--yellow); outline-offset: 3px; }
  .idea .go {
    font-family: var(--font-display); font-weight: 700; font-size: .78rem; text-transform: uppercase;
    letter-spacing: .05em; color: var(--violet); white-space: nowrap;
  }
  @media (max-width: 560px) { .idea { grid-template-columns: 34px 1fr; } .idea .go { grid-column: 2; } }
  @media (prefers-reduced-motion: reduce) { .idea { transition: none; } .idea:hover { transform: none; } }

  .rotate {
    display: none; position: fixed; inset: 0; z-index: 100; background: var(--bg); color: var(--fg);
    place-content: center; justify-items: center; gap: 16px; text-align: center;
    padding: max(24px, env(safe-area-inset-top, 0px)) max(20px, env(safe-area-inset-right, 0px)) max(24px, env(safe-area-inset-bottom, 0px)) max(20px, env(safe-area-inset-left, 0px));
  }
  .rotate h2 { font-size: clamp(1.7rem, 8vw, 2.4rem); color: var(--violet); text-transform: uppercase; }
  .rotate p { max-width: 28ch; color: var(--muted); }
  .rotate svg { width: 120px; height: 120px; color: var(--teal); animation: girar 2.8s ease-in-out infinite; }
  @keyframes girar { 0%, 20% { transform: rotate(0deg); } 60%, 100% { transform: rotate(-90deg); } }
  @media (prefers-reduced-motion: reduce) { .rotate svg { animation: none; transform: rotate(-90deg); } }
  @media (orientation: portrait) and (max-width: 900px) {
    body:not(.skip):not(.turned) .rotate { display: grid; }
    body:not(.skip):not(.turned) .scroller { display: none; }
  }
  @media (orientation: landscape) and (max-height: 520px) {
    .poster { padding-block: 26px 20px; gap: 18px; }
    .top h1 { font-size: clamp(1.7rem, 5vw, 2.4rem); }
    .grid { gap: 22px; }
  }

  .sheet {
    display: none; position: fixed; inset: 0; z-index: 200; background: rgba(10, 20, 22, .65);
    align-items: center; justify-content: center;
    padding: max(10px, env(safe-area-inset-top, 0px)) max(12px, env(safe-area-inset-right, 0px)) max(10px, env(safe-area-inset-bottom, 0px)) max(12px, env(safe-area-inset-left, 0px));
  }
  .sheet.open { display: flex; }
  .panel {
    width: min(620px, 100%); max-height: 100%; overflow: auto; background: var(--panel); color: var(--fg);
    border: 4px solid var(--frame); border-radius: 16px; padding: 16px 18px 18px; display: grid; gap: 10px;
  }
  .panel header { display: grid; grid-template-columns: auto 1fr auto; gap: 12px; align-items: center; }
  .panel .big {
    font-family: var(--font-display); font-weight: 700; font-size: 1.6rem; line-height: 1;
    background: var(--teal); color: var(--on-teal); width: 44px; height: 44px; border-radius: 50%;
    display: grid; place-items: center;
  }
  .panel h2 { font-size: 1.35rem; color: var(--violet); text-transform: uppercase; }
  .panel textarea {
    width: 100%; font: inherit; color: var(--fg); background: var(--surface);
    border: 2px solid var(--line); border-radius: 10px; padding: 10px 12px; resize: vertical; min-height: 92px;
  }
  .panel textarea:focus-visible { outline: 3px solid var(--yellow); outline-offset: 2px; }
  .panel .row { display: flex; flex-wrap: wrap; gap: 10px; }
  .partial { font-style: italic; color: var(--muted); font-size: 15px; min-height: 1.2em; }
  .status { min-height: 1.4em; font-weight: 600; }
  .status.ok { background: #d6f3e8; color: #0d4a3f; padding: 6px 10px; border-radius: 8px; }
  .status.err { background: #fde2df; color: #7a1a12; padding: 6px 10px; border-radius: 8px; }
  .note { font-size: 14px; color: var(--muted); }
  .mic.listening { background: #e0453a; border-color: #e0453a; color: #fff; }
  .inbox { display: grid; gap: 10px; }
  .msgcard { background: var(--surface); border-radius: 10px; padding: 10px 12px; display: grid; gap: 4px; }
  .msghead { font-size: 13px; font-weight: 700; color: var(--muted); }
  .msgcard p { white-space: pre-wrap; overflow-wrap: anywhere; }
  .adminbar { max-width: 1060px; margin: 16px auto 0; display: flex; justify-content: center; }

  .stage { display: contents; }
  .turnbar { display: none; }
  @media (orientation: portrait) and (max-width: 900px) {
    body.turned { overflow: hidden; }
    body.turned .stage {
      display: block; position: fixed; top: 0; left: 0; overflow: hidden;
      width: 100vh; width: 100dvh; height: 100vw;
      transform-origin: 0 0; transform: rotate(90deg) translateY(-100%);
    }
    body.turned.flip .stage { transform: rotate(-90deg) translateX(-100%); }
    body.turned .scroller {
      position: absolute; inset: 0; overflow: auto; -webkit-overflow-scrolling: touch;
      padding: calc(14px + env(safe-area-inset-right, 0px)) calc(14px + env(safe-area-inset-bottom, 0px)) calc(20px + env(safe-area-inset-left, 0px)) calc(14px + env(safe-area-inset-top, 0px));
    }
    body.turned.flip .scroller {
      padding: calc(14px + env(safe-area-inset-left, 0px)) calc(14px + env(safe-area-inset-top, 0px)) calc(20px + env(safe-area-inset-right, 0px)) calc(14px + env(safe-area-inset-bottom, 0px));
    }
    body.turned .turnbar { display: flex; flex-wrap: wrap; gap: 10px; justify-content: center; margin: 0 auto 12px; }
    body.turned .grid { grid-template-columns: 1.1fr 1fr; }
    body.turned .arrow { display: block; }
    body.turned .idea { grid-template-columns: 34px 1fr auto; }
    body.turned .idea .go { grid-column: auto; }
    body.turned .poster { padding-block: 26px 20px; gap: 18px; }
    body.turned .top h1 { font-size: 2.4rem; }
    body.turned .top .sub { font-size: 1.15rem; }
    body.turned .why h2 { font-size: 1.5rem; }
    body.turned .closing { font-size: 1.9rem; }
  }
</style>
</head>
<body>

<div class="rotate">
  <svg viewBox="0 0 64 64" fill="none" stroke="currentColor" stroke-width="3.5" stroke-linecap="round" stroke-linejoin="round" aria-hidden="true"><rect x="19" y="6" width="26" height="52" rx="5"/><path d="M28 51h8"/></svg>
  <h2>Turn your phone</h2>
  <p>Turn your phone sideways and tap "I turned it". The poster turns with it.</p>
  <button class="btn" type="button" id="turn">I turned my phone</button>
  <button class="btn alt" type="button" id="skip">Keep it upright</button>
</div>

<div class="stage"><div class="scroller">
<div class="turnbar">
  <button class="btn alt" type="button" id="flip">Flip to the other side</button>
  <button class="btn alt" type="button" id="unturn">Upright view</button>
</div>

<div class="frame">
  <main class="poster">
    <header class="top">
      <div class="tag">
        <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" aria-hidden="true"><path d="M12 3l8 3v6c0 4.5-3.2 8-8 9-4.8-1-8-4.5-8-9V6l8-3z"/><path d="M8.5 12l2.5 2.5 4.5-5"/></svg>
        Safe Community <span>| Volunteer group</span>
      </div>
      <h1>Let's fight crime!</h1>
      <p class="sub">A safe community depends on all of us!</p>
    </header>

    <div class="grid">
      <section aria-labelledby="ideias">
        <h2 class="eyebrow" id="ideias">What can we do? Here are three ideas:</h2>
        <p class="hintline">Tap an idea to speak or write your message.</p>
        <ol class="ideas">
          <li><button class="idea" type="button" data-i="1"><span class="n">1</span><span><b>Report it:</b> tell the authorities about criminal activity.</span><span class="go">Speak or write</span></button></li>
          <li><button class="idea" type="button" data-i="2"><span class="n">2</span><span><b>Respect the law:</b> respect people and everyone's rights.</span><span class="go">Speak or write</span></button></li>
          <li><button class="idea" type="button" data-i="3"><span class="n">3</span><span><b>Help your community:</b> take part in awareness and violence-prevention campaigns.</span><span class="go">Speak or write</span></button></li>
        </ol>

        <figure class="mapfig" aria-label="Illustration of the neighborhood with the plaza highlighted">
          <div class="map">
            <svg viewBox="0 0 420 270" role="img" aria-label="Illustrated aerial view of a neighborhood with trees, streets and a red marker">
              <rect width="420" height="270" fill="#7fb36f"/>
              <g fill="#3f8f4a">
                <circle cx="30" cy="40" r="16"/><circle cx="62" cy="22" r="12"/><circle cx="50" cy="86" r="14"/><circle cx="20" cy="140" r="18"/>
                <circle cx="70" cy="200" r="15"/><circle cx="28" cy="236" r="13"/><circle cx="120" cy="250" r="14"/><circle cx="170" cy="30" r="13"/>
                <circle cx="215" cy="22" r="11"/><circle cx="380" cy="30" r="16"/><circle cx="400" cy="90" r="13"/><circle cx="392" cy="170" r="17"/>
                <circle cx="350" cy="250" r="14"/><circle cx="300" cy="256" r="11"/>
              </g>
              <g fill="#2f7a3d">
                <circle cx="46" cy="60" r="9"/><circle cx="100" cy="228" r="9"/><circle cx="198" cy="52" r="8"/><circle cx="372" cy="62" r="9"/><circle cx="410" cy="206" r="9"/>
              </g>
              <g transform="rotate(-16 210 135)">
                <g stroke="#8e9aa6" stroke-width="12" stroke-linecap="butt">
                  <path d="M-40 130 H470"/><path d="M130 -40 V310"/><path d="M300 -40 V310"/>
                </g>
                <g stroke="#f4f4f0" stroke-width="1.5" stroke-dasharray="8 8" opacity=".8">
                  <path d="M-40 130 H470"/><path d="M130 -40 V310"/><path d="M300 -40 V310"/>
                </g>
                <g fill="#dfe6ee">
                  <rect x="150" y="30" width="62" height="38" rx="3"/><rect x="222" y="30" width="62" height="38" rx="3"/>
                  <rect x="150" y="76" width="62" height="38" rx="3"/><rect x="222" y="76" width="62" height="38" rx="3"/>
                  <rect x="320" y="30" width="60" height="38" rx="3"/><rect x="320" y="76" width="60" height="38" rx="3"/>
                  <rect x="150" y="146" width="62" height="40" rx="3"/><rect x="222" y="146" width="62" height="40" rx="3"/>
                  <rect x="320" y="146" width="60" height="40" rx="3"/><rect x="320" y="196" width="60" height="40" rx="3"/>
                  <rect x="60" y="30" width="56" height="38" rx="3"/><rect x="60" y="76" width="56" height="38" rx="3"/>
                  <rect x="150" y="196" width="62" height="40" rx="3"/>
                </g>
                <g fill="#3c6fb5">
                  <rect x="158" y="38" width="20" height="10"/><rect x="230" y="84" width="20" height="10"/><rect x="328" y="38" width="20" height="10"/>
                  <rect x="158" y="154" width="20" height="10"/><rect x="330" y="204" width="20" height="10"/>
                </g>
              </g>
              <path d="M250 112c-14 0-24 10-24 24 0 18 24 40 24 40s24-22 24-40c0-14-10-24-24-24z" fill="#e0453a"/>
              <circle cx="250" cy="136" r="9" fill="#fff"/>
              <path d="M262 190 Q292 238 338 236" fill="none" stroke="#e0453a" stroke-width="8" stroke-linecap="round"/>
              <path d="M330 220 L352 238 L328 252 Z" fill="#e0453a"/>
            </svg>
          </div>
          <figcaption class="maplabel">Our neighborhood</figcaption>
          <div class="zoom">
            <svg viewBox="0 0 200 200" role="img" aria-label="Plaza with people gathered">
              <rect width="200" height="200" fill="#67b05a"/>
              <circle cx="100" cy="100" r="48" fill="none" stroke="#e3d3ac" stroke-width="12"/>
