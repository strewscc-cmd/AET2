# Harapan High Squad — 3-Period Classroom Runbook

Duration: 3 lesson periods × 45 minutes = 135 minutes.
Audience: Grade XI, Analytical Exposition (B2 target).
Core game: existing nine-mission Harapan High Squad. Add-ons are optional side activities and must not block the main missions.

## Learning outcomes
Students can:
1. identify common logical fallacies in claims;
2. defend a position with a clear claim, relevant evidence and reasoning;
3. present a counterargument and respond respectfully;
4. distinguish a strong argument from an emotional or unsupported statement.

## Recommended 135-minute flow

### 0–10 min — Briefing and launch
- Teacher explains the mission, class rules, and respectful disagreement.
- Students log in and enter their assigned room.
- Remind students that the game missions remain the main progression; side challenges are bonus learning stops.

### 10–35 min — Main missions: exploration round
- Students explore the school map and start the nine existing missions.
- Teacher circulates and checks that students understand thesis, arguments, evidence and reiteration.
- Students who finish a mission early may open the new Challenges button.

### 35–55 min — 🔍 Find the Fallacy
- Students open Challenges → Find the Fallacy.
- Each student completes one or more short cases.
- After selecting an answer, students must explain why the argument is faulty to a partner or teacher.
- Teacher follow-up prompts: What evidence is missing? Is the sample representative? Is the speaker attacking the person or the idea?
- Suggested group extension: each team writes one flawed argument and asks another team to identify the fallacy.

### 55–80 min — 🎭 Defend Your Opinion
- Students open Challenges → Defend Your Opinion.
- They choose the strongest defence for a position they may personally disagree with.
- Pair discussion: one student supports the position; the other gives a counterargument; then they switch roles.
- Require a reason + evidence/example + explanation, not merely “I agree”.
- The in-game Defend gate also appears before a steal attempt. Treat this as a game mechanic, not a replacement for the speaking task.

### 80–105 min — Main missions and canteen break
- Students return to the nine core missions.
- They can visit the canteen and use the added menu items. Food restores energy according to the existing game economy; it does not grant XP.
- Teacher can pause the class for a 3-minute stretch/water break around the midpoint.

### 105–125 min — Team argument battle
- Teacher assigns each team a motion related to school life (phone restrictions, school meals, uniform rules, homework, or school waste).
- Teams prepare a 60-second defence with one claim, two reasons and one piece of evidence/example.
- Opposing team asks one challenge question. Students must defend a view fairly, even if it is not their personal opinion.

### 125–135 min — Reflection and exit ticket
Students answer:
1. Which fallacy was easiest to recognise? Why?
2. What makes evidence relevant and convincing?
3. What did you learn from defending an unfamiliar position?
4. Write one thesis statement and one supporting argument.

## Keep the session lively
- Alternate individual game play, pair discussion, team competition and short movement breaks.
- Do not force every student to complete every side challenge; use them as bonus stations.
- Avoid public shaming for incorrect answers. Ask students to explain and revise their reasoning.
- Rotate the spokesperson so the same confident students do not dominate.

## Current in-game add-ons
Implemented in this release:
- Side Challenge Hub button in the game.
- Find the Fallacy: randomized multiple-choice cases with feedback; correct answer grants +10 XP.
- Defend Your Opinion: randomized argument-selection cases with feedback; correct answer grants +10 XP.
- Defend gate before the existing steal action. Failed defence cancels the steal attempt without changing money or XP.
- Six additional canteen items, appended to the existing menu; all preserve the original price/energy-restoration mechanic.

Not yet implemented:
- Challenge Another online player duel.
- Shared room-wide Think Fast every five minutes.
- Random-interval Mystery Food Gacha and cross-subject MBG rescue quiz.
- Swap-position defence hook: no swap-position handler was found in the inspected student game file, so no speculative integration was added.

## Technical safety and next release
The current game uses browser-side JavaScript and Firebase REST. A secure shared timer, fair duel adjudication, random gacha draw and tamper-resistant reward system require a trusted backend (for example Firebase Cloud Functions) and suitable database rules. Do not treat browser-only random timers or client-calculated winners as tamper-proof. These features should be added after backend coordination is available.
