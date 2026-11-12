# Harapan High Squad — Add-on Technical Specification

Status: design baseline, not yet integrated into gameplay.
Target branch: main
Current inspected files:
- index.html — blob d75dda44f05a7b26fdb37ae3ef13264d325ef479
- teacher.html — blob 0ec63d6cc6a4b193f79116a73da17f9736afcd17

## Non-negotiable compatibility rules
1. Do not replace the existing login, room assignment, role assignment, nine-mission flow, player save format, certificate, teacher end-session flow, or existing reward variables.
2. Existing player records remain under /hh/{room}/{playerId}. New state lives only under /hh_addons/{room}/...
3. Keep the current FOOD entries and their semantics (price in rupiah; e restores energy). Add new menu entries using the same shape {n, ic, p, e}; do not silently change existing prices or energy values.
4. Every event and challenge is scoped to one room.
5. Every reward/penalty must be idempotent (same event/player cannot be applied twice).
6. If add-on data is missing or unavailable, the core game continues normally.

## Confirmed game facts from the inspected main branch
- Core player state is local-first and saved in localStorage key hh_save_v1.
- Multiplayer uses Net peers and Firebase REST paths /hh/{room}, /hh_ev/{room}/{playerId}; cloud polling runs approximately once per second.
- Existing peer freshness is inferred from heartbeat timestamps. Teacher dashboard classifies players online when their heartbeat age is <= 5 seconds, idle at <= 15 seconds, and offline after that.
- Existing steal flow is in stealTo()/onEv(): the attacker sends a steal event; the target currently processes the loss and sends a response. A defensive challenge must be completed BEFORE stealTo() sends the event, and the target must not be debited until the attacker has passed the challenge.
- Existing canteen FOOD list contains Es Teh, Cimol Bojot, Bakso, and Nasi Uduk Komplit. Purchases deduct money, increment snackCount, and restore energy.
- No swap-position handler was found in the inspected index.html. Do not wire a speculative hook. Locate or define the actual swap feature before integrating Defend Your Opinion with it.
- Student client currently polls teacher controls every 2 seconds; this is separate from the one-second game network poll.

## Add-on modules

### A. Add-on bootstrap
Suggested future files:
- addons/hhs-addon.js
- addons/hhs-addon.css

The bootstrap should expose one namespaced object only, e.g. HHSAddon, and receive narrow adapters from the existing game:
- getPlayer(): current player identity and room
- getOnlinePeers(): fresh peers in the current room
- getStats()/applyReward(): existing XP and money state via the existing HUD/persist functions
- showModal()/toast(): existing UI helpers
- sendRoomEvent()/subscribeRoomEvents(): room-scoped event transport

Avoid creating a second player model or a second save system. Do not overwrite global functions.

### B. Challenge Another
- Unlock thresholds: 100, 250, 500, 750 XP. A threshold can be claimed once per player per run/session; store claims in add-on state.
- Player opens a list of fresh online peers from the same room only; exclude self and completed/ended players.
- Invite has a 20-second acceptance window.
- One active duel per player; prevent multiple simultaneous invites.
- Both players receive the same questionId, options, and deadline from the duel record.
- Suggested answer window: 30 seconds.
- Winner rules: correct beats incorrect; if both correct, faster server-received answer wins; if both wrong or deadline expires without a valid answer, draw.
- Suggested reward: winner +25 XP and +Rp15,000; loser receives no penalty. Apply once using a unique duelId.
- Do not use local-only time as authoritative ordering. Server receipt timestamps or a trusted coordinator must decide the winner.

### C. Think Fast
- One event per room every 5 minutes, not one independent timer per student.
- A room event record contains eventId, questionId, startsAt, endsAt, status, and room.
- Suggested answer window: 20 seconds; one attempt per player; correct answer +10 XP; incorrect/timeout no penalty.
- All clients render the same active event and question.
- Deduplicate answers and rewards by eventId + playerId.
- A browser-local setInterval may render countdown only; it must not independently create authoritative events.

### D. Mystery Food Gacha
- Random event interval: choose next delay uniformly between 8 and 15 minutes after the previous event closes.
- One gacha event per room; all clients see the same event window.
- Each player can claim/open once per event.
- Initial proposed probabilities:
  - MBG Biasa 45%: +10 XP
  - MBG Spesial 25%: +20 XP and +Rp10,000
  - MBG Sultan 10%: +50 XP
  - MBG Misterius 10%: random non-poison reward
  - MBG Beracun 10%: cross-subject rescue quiz
- For MBG Beracun: answer one Mathematics, Chemistry, or Physics question. Correct answer neutralizes the poison and grants +20 XP; incorrect/timeout applies -40 XP.
- Use a single server/coordinator-generated outcome per player/event and store it before displaying it, so refresh cannot reroll the gacha.
- Keep the probability table configurable by teacher. Validate total probability = 100.

### E. Defend Your Opinion
- Before an existing steal action, show one short Analytical Exposition question.
- If correct, proceed with the existing steal flow unchanged.
- If incorrect or timed out, cancel the attempted action without changing either player's money.
- Do not penalize XP for a failed defense.
- For swap, integration is blocked until a real swap handler is identified in the current codebase. Once found, call the same defense gate before the swap mutation.
- Challenge attempt must be bound to a unique actionId to prevent replay.

### F. Canteen extension
- Append new menu entries without deleting or editing existing FOOD entries.
- Preserve the existing data contract {n, ic, p, e}, with p in rupiah and e as energy restored.
- Initial suggested additions (prices/effects are balancing proposals, not final committed economy values):
  - Mie Jebew 🍜: Rp10,000 / +35 energy
  - Matcha Latte 🍵: Rp12,000 / +25 energy
  - Brown Sugar Boba 🧋: Rp12,000 / +25 energy
  - Nasi Goreng 🍳: Rp15,000 / +55 energy
  - Cireng 🥟: Rp5,000 / +15 energy
  - Ayam Geprek 🍗: Rp18,000 / +65 energy
- Do not add XP bonuses to ordinary food purchases in the first implementation; this avoids changing the XP economy. Special temporary food buffs can be a later optional extension.

## Firebase namespace proposal
All paths are additive and room-scoped:
- /hh_addons/{room}/config
- /hh_addons/{room}/events/{eventId}
- /hh_addons/{room}/duels/{duelId}
- /hh_addons/{room}/claims/{eventId}/{playerId}
- /hh_addons/{room}/rewards/{eventId}/{playerId}
- /hh_addons/{room}/presence/{playerId} (only if existing heartbeat cannot safely support the online list)

Do not store private answer keys in a path readable by every student. A trusted backend should validate answers and write final outcomes.

## Authority and security requirement
The existing game uses browser clients and Firebase REST. A browser-only implementation cannot securely guarantee fair duel timing, prevent modified clients from awarding themselves XP, or safely run a room-wide random event coordinator. Recommended production path: Firebase Cloud Functions (or another trusted server) for event creation, answer validation, gacha outcomes, and reward writes. A lower-security classroom prototype can use a single elected room coordinator with Firebase ETag/conditional writes, but it must be labelled prototype-only and must not be treated as tamper-proof. Never weaken the existing database rules to make add-ons work.

## Implementation order and acceptance tests
1. Add isolated bootstrap and menu extension only. Verify old food still works and existing progress/save/certificate remain intact.
2. Add gacha event and per-player claim. Verify reload cannot reroll; other rooms do not see the event.
3. Add Defend Your Opinion gate to steal. Verify failed defense never changes money; successful defense triggers existing steal flow exactly once.
4. Identify/implement swap only after confirming the intended swap mechanic.
5. Add Think Fast with one shared room event and deduplicated rewards.
6. Add Challenge Another with online-only roster and server-ordered results.

Before release, test: two students in same room; students in two different rooms; one student offline; simultaneous claims; refresh during event; duplicate event delivery; network loss; end-game during an event; teacher disabling add-ons; and old saved players continuing without a migration.
