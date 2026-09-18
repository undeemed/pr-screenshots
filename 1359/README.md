# PR 1359 — rebase capture evidence

Before/after captures for
<https://github.com/The-Swarm-Corporation/swarms-platform/pull/1359> after its
rebase, plus the measurements behind them. Images live in
`undeemed/pr-screenshots` (RULES.md 2.5 — no binaries in this repository).

- before: `main` at `3d1f7fe5`
- after: branch `fm/swarms-chat-mobile-1330` at `12ccb0bf`
- surface: `/prompt/b0000000-0000-4000-8000-000000000001/chat`, anonymous, dev
  server on the shared local Supabase fixture

## Captures

Every capture measured `window.innerWidth` in the same evaluation that took the
screenshot; all ten reported the requested width.

| Subject | Width | Before | After |
|---|---|---|---|
| Prompt chat | 1280 | [before](https://raw.githubusercontent.com/undeemed/pr-screenshots/master/1359/01-prompt-chat-1280-before.png) | [after](https://raw.githubusercontent.com/undeemed/pr-screenshots/master/1359/01-prompt-chat-1280-after.png) |
| Prompt chat | 390 | [before](https://raw.githubusercontent.com/undeemed/pr-screenshots/master/1359/02-prompt-chat-390-before.png) | [after](https://raw.githubusercontent.com/undeemed/pr-screenshots/master/1359/02-prompt-chat-390-after.png) |
| Agent settings dialog | 1280 | [before](https://raw.githubusercontent.com/undeemed/pr-screenshots/master/1359/03-settings-dialog-1280-before.png) | [after](https://raw.githubusercontent.com/undeemed/pr-screenshots/master/1359/03-settings-dialog-1280-after.png) |
| Agent settings dialog | 390 | [before](https://raw.githubusercontent.com/undeemed/pr-screenshots/master/1359/04-settings-dialog-390-before.png) | [after](https://raw.githubusercontent.com/undeemed/pr-screenshots/master/1359/04-settings-dialog-390-after.png) |
| Model picker sheet | 390 | [before](https://raw.githubusercontent.com/undeemed/pr-screenshots/master/1359/05-model-picker-390-before.png) | [after](https://raw.githubusercontent.com/undeemed/pr-screenshots/master/1359/05-model-picker-390-after.png) |

## Does the desktop settings dialog stay compact?

Yes. Measured `getBoundingClientRect().height` and computed `font-size` of the
dialog's visible inputs:

| Ref | 1280px | 390px |
|---|---|---|
| `main` 3d1f7fe5 | 28/28/28/28/40 px, all 11px text | 28/28/28/28/40 px, all 11px text |
| branch 12ccb0bf | 28/28/28/28/40 px, all 11px text | 44/44/44/44/54 px, all 16px text |

`h-7` is 28px and `md:text-[11px]` is 11px, so the inlined
`min-h-[44px] … md:h-7 md:min-h-0 md:text-[11px]` reproduces main's desktop
values exactly and only grows below the `md` breakpoint.

Both 1280px pairs are byte-identical (`01` md5 `c8f429908538c366bc8f0a17cd98abc9`,
`03` md5 `48b72d5f324804f6b9a65d7df20c07b8`), while all three 390px pairs differ.
The same two server runs produced differing phone captures, so the desktop
identity is a property of the code, not a stale build.

## Model picker overlay versus the fixed navbar

The navbar is covered in both refs, so this pair does not show a
peeking-navbar regression being fixed.

`document.elementFromPoint` at y=28 across x = 30, 120, 195, 300, 360 returns the
sheet overlay, never a navbar descendant, on `main` as well: main's own
`fixed inset-0 z-[10001] bg-black/50` already paints above the navbar's
`z-index: 9999`. The branch swaps that hand-rolled overlay for the shared
`Drawer`, whose overlay is `bg-black/80` inside a new `relative z-[10001]`
portal host, so the navbar strip is darker after (sampled navbar pixels drop
from ~50% to ~20% of their unobscured value).

## Checks

| Command | Result |
|---|---|
| `pnpm exec tsc --noEmit` on 12ccb0bf | 109 `error TS` lines, 14 files |
| `pnpm exec tsc --noEmit` on 3d1f7fe5 | 109 `error TS` lines, identical output (`diff` empty) |
| `pnpm lint` on 12ccb0bf | exit 0; one pre-existing warning in `app/launch/page.tsx` |

No error names a file under `shared/components/chat/` on either ref, so the
branch adds no type errors. The failures are pre-existing and unchanged.

## Not done

- No production capture; the Vercel deployment check on 1359 is failing and
  production sits behind the firewall (RULES.md 8.3).
- Anonymous session only. The chat surface renders without a signed-in user;
  the logged-in variant was not captured.
- `chrome-devtools-axi resize` reports success but leaves `innerWidth` at 1280
  on this host, so captures were taken over CDP with
  `Emulation.setDeviceMetricsOverride` and the width asserted per capture.
