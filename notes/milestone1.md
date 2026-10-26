# Milestone 1 — Community and candidate distinctions

## Community

**Hacker News comments** (news.ycombinator.com). Public, no login needed to
read, almost entirely text, and quality varies a lot within a single thread.
Not tied to one show or release, so no spoilers.

Reddit was the first choice (r/mildlyinteresting, then r/nyu), but Reddit
blocks automated reads from this environment (403), and r/mildlyinteresting
is mostly photo titles with little opinion to sort.

## What I read

48 comments, pulled 2026-10-26 through the public HN Algolia API, from six
front-page threads: software ambition, a CNN vs. LLMs for skin cancer,
Dolly Parton's Imagination Library, internet censorship / FOSDEM, the
Brazilian election, and LLM price cuts. The reading was done with Claude
(AI-assisted). This is disclosed in the README.

## Candidate distinctions

**1. Some comments cite something checkable. Most don't.**
A date, a number, a link, a named precedent. Others assert without support.
- Cites: *"The US only enacted term limits in 1951. Before that time, only two
  presidents had attempted to seek third terms..."*
- Cites: *"Reminds me a bit of how Andrew Carnegie funded building public
  libraries... https://en.wikipedia.org/wiki/Carnegie_library"*
- Doesn't: *"They are too big to implode."*
- Doesn't: *"it is a language model after all"*

**2. Some comments are one-line quips. Others build an argument over several sentences.**
The quips are short, often sarcastic, and add no reasoning.
- Quip: *"Sadly, my GPUs were lost in a tragic boating accident."*
- Quip: *"Are you unsure if he is evil?"*
- Argument: *"Censorship is when Mallory decides that Alice is not permitted to
  talk to Bob. Moderation is when Bob does not want to hear from Alice and
  Mallory blocks her for him..."*

**3. Some comments rest on the commenter's own experience instead of general claims.**
- *"All of the books I've received through this program have been paid for by
  the local library and other city funds..."*
- *"I use Kimi, chatGPT, and Claude, and Kimi is by far the most expensive in
  terms of tokens and $/work done."*
- *"I can see that mine opened and closed at the expected time."*

**4. Many comments quote a line from a parent (`> ...`) and rebut it.**
These are structurally easy to spot. The open question is whether the
rebuttal is supported (distinction 1) or just contradiction: *"> much
cheaper to access — I'm calling BS."*

## Early thoughts for Milestone 2

- Distinction 1 (checkable support) looks like the strongest label boundary.
  It points at something visible in the text.
- Distinction 2 (quip vs. argument) is common and clean at the extremes. The
  middle, a two-sentence opinion with no evidence, is where it will blur into
  a "claim with no support" label.
- Distinction 3 (personal experience) overlaps with 1. Does "I measured X on
  my machine" count as checkable? That is a likely hardest boundary.
- Distinction 4 is about structure, not substance. It is probably a feature
  rather than a label.
