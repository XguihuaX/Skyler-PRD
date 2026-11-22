> 旧 MomoOS-v2 项目材料快照；其中实施状态和路径按原工作区理解。新需求范围见 [主 PRD](docs/product/PRD-momo-next.md)，执行顺序见[任务清单](docs/product/SKYLER-TASKS.md)。

# Skyler Design Lite

This is the short design note for the current development direction. It is not a
marketing document and it does not replace source code.

Status vocabulary in this file is deliberate:

- **published baseline** means the code exists on the current GitHub `main`;
- **local validation batch** means implementation and automated evidence exist
  in the maintainer's working tree, but the code has not yet been published as
  one reviewed implementation batch;
- **planned** means design only.

## Project shape

Skyler / Momo is a personal desktop assistant with Live2D and character-based
interaction. The current user goal is personal daily use with commercial product
quality. Social character state remains a supported capability.

As of 2026-11-17, the active architecture review compares evolving the custom
runtime, replacing the execution core while retaining product capabilities, and
rebuilding the whole product. [ARCH-001](docs/product/ARCH-001-runtime-renewal.md)
records the earlier recommendation for the second route; no new framework has
been selected or installed. The user's subsequent decision is to define the new
product top down: functions, requirements and acceptance, technical selection,
detailed design, then Astra implementation. The current entry is
[the new product PRD](docs/product/PRD-momo-next.md). Existing UI and components
are reuse candidates, not constraints on the new design. The earlier
character-state-first sequence no longer gates assistant work.

The project is built around:

- structured Persona;
- runtime character state;
- memory and activity context;
- Live2D + voice expression;
- desktop presence through a main panel and a floating window;
- tools exposed through a capability layer.

## Main design principle

The character should not be rebuilt from a prompt on every turn.

A turn should have access to:

- who the character is;
- what kind of card it is;
- how the character is currently doing;
- what has happened recently;
- what the user is asking now;
- what tools or screen evidence are available.

This is why Persona, `CharacterState`, memory, activity context, and capabilities
are separate layers instead of one large prompt string.

## UI forms

Skyler has two desktop forms.

### Main panel

The main panel is for deeper interaction:

- full chat;
- history;
- character details;
- Persona editing;
- provider and capability settings;
- Live2D management.

### Desktop-pet window

The floating window is for low-friction presence:

- transparent Live2D window;
- idle animation;
- brief message bubbles;
- voice / VAD state;
- quick controls.

The two forms share the same character, state, and context. They should not be
treated as separate character instances.

## Persona and card type

Persona is structured because a single prompt blob is hard to maintain.

Current Persona fields include Tier-1 style fields such as identity,
personality core, speech style, relationship, voice samples, and forbidden
phrases. Extended fields include taboo topics, lore, and capability overrides.

`card_type` is the first split between social cards and assistant cards.

- Social cards lean toward companionship, relationship, and emotional expression.
- Assistant cards lean toward task positioning, tool capability, and behavior
  boundaries.

The schema, API, editor path, and prompt path are connected. Full runtime policy
and lifecycle separation is still in progress. The target boundary is a
resident assistant card for tools, tasks, OS context, notifications, and
controlled actions, with social cards loaded on demand for Persona,
relationship, private life, character state, and companion expression.

## Character state

The published baseline stores runtime state such as:

- mood;
- intimacy;
- current thought;
- current activity;
- last interaction time.

The local FSM v1 validation batch extends this with energy, emotional valence,
arousal, transient events, deterministic settlement, bounded change, decay, and
restart catch-up.

The current design no longer lets the model directly submit final mood, energy,
valence, arousal, activity, or intimacy values. A social character may emit a
controlled `<state_event>` describing a real interaction event; program logic
validates the event and settles the resulting state. Assistant cards do not
receive this fictional private-life state machine.

FSM v1 affects current expression through a natural derived state. It does not
choose future DailyAgent activities, rewrite the daily plan, or automatically
change Persona. Mood-aware flexible behavior selection is a later, separate
layer.

## Memory and context

The current memory/context stack should be described by its real sources:

- short-term conversation window;
- conversation summary;
- long-term semantic memory;
- user profile;
- activity context;
- current character state.

It is okay to call this "four-layer memory" in demo wording, but the detailed
docs should name the actual sources above. Do not invent a separate memory
architecture that is not in code.

## Activity context

ActivityWatcher records what the user has recently been doing on the computer:

- foreground app;
- browser URL/title;
- document path;
- activity sessions.

ActivityTimeline stores this locally in `activity_sessions` and can format a
daily activity summary for the ChatAgent prompt.

This gives the character background context. It is different from reading the
current screen structure.

## Layered Desktop Perception

The current desktop perception design is layered.

### Built

- ActivityWatcher and ActivityTimeline provide recent activity context.
- `screen.read_current_screen` reads the current foreground app's AX / UI Tree
  summary on demand.
- The current default is intentionally narrow: foreground app, read-only,
  on-demand.

This default exists to control token use, latency, and privacy risk. Skyler
should not continuously read every window by default.

### In progress

The next layer is:

- Window Roster: maintain an app / window / PID roster.
- Watchlist: maintain user-authorized or context-relevant targets.
- On-demand Deep Read: if a target still exists, changes, or the user asks about
  it, read that target PID with AX / UI Tree.

If structured UI information is not enough, the later fallback is screenshot or
local-crop visual understanding. That is not the same thing as user-uploaded
image input.

## DailyAgent

The published baseline contains Stage 1:

- daily plan generation;
- current-activity ticker;
- writing current activity back into character state.

The local Stage 2A validation batch adds:

- per-character world anchors, time zones, cities, and private scene labels;
- real-weather snapshots with omission instead of invented fallback facts;
- structured schedule blocks, current/next activity, elapsed and remaining
  time, restart recovery, and social-card isolation;
- separate base schedules and temporary overrides;
- event-level activity refinement shortly before an event starts.

DailyAgent continues to generate coarse events. Refinement maps a coarse event
through `activity_type → activity_kind → variant → controlled detail → phase
plan`. Phase completion is calculated from persisted real-time boundaries, not
from an LLM claim. The current acceptance gate is real conversation testing of
short-interval activity continuity.

DailyAgent is still not a complete autonomous-life engine: adaptive replanning,
travel and movement rules, multi-character budgets, and mood-aware flexible
activity choice remain later work.

## Capability layer

Capabilities are registered through a common registry instead of being hardcoded
inside the chat loop.

Current pieces:

- `CapabilityRegistry`;
- native Python capabilities;
- MCP client for external tools;
- MCP server for exposing Skyler capabilities;
- provider configuration;
- per-tool switches;
- confirm-gate framework.

Dangerous tools and write actions still need end-to-end validation. The design
should assume that any mutating action requires explicit confirmation and
credential governance.

## Voice and Live2D

The current voice chain covers:

- VAD;
- ASR;
- LLM streaming;
- TTS;
- Live2D lipsync / expression hooks.

Emotional TTS can use character state. Live2D motion selection and a full action
library are future work, not current capability.

## What not to overstate

Do not describe these as finished:

- the local validation batch as already published on GitHub;
- autonomous or mood-directed DailyAgent behavior beyond Stage 2A;
- full social / assistant runtime split;
- continuous VLM screen monitoring;
- reading all windows in the background;
- write-action safety fully verified end to end;
- multi-character collaboration;
- long-term autonomy;
- Live2D motion planning;
- Anime asset generation;
- role-facing ComfyUI workflow integration. The local connector is isolated
  infrastructure only.

## Code vocabulary

Use these terms consistently:

- Persona
- card_type
- CharacterState
- ActivityWatcher
- ActivityTimeline
- Layered Desktop Perception
- Window Roster
- Watchlist
- On-demand Deep Read
- CapabilityRegistry
- MCP
- Confirm Gate
