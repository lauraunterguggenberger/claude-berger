# decision-atoms — origin

This skill was built from a single prompt. If you just want the original and not the
skill wrapped around it, it is reproduced verbatim below.

## The original prompt

> Convert this into two sections only. First, an atomic fact table with IDs: facts,
> assumptions, constraints, and unresolved unknowns. Second, a composition showing only
> the minimum logical relationships between those IDs required to reach the decision. Do
> not summarize the prose. Do not repeat facts in the composition. Preserve uncertainty.
> Show the flip condition that would change the decision.

Paste that above any decision document and it works reasonably well on its own. `SKILL.md`
is what it became after testing.

## What changed between the prompt and the skill, and why

The prompt states four requirements as prohibitions: *do not summarize, do not repeat
facts, preserve uncertainty, show the flip condition*. Testing showed prohibitions are the
wrong form for this failure.

Three baseline runs with no guidance produced no addressable IDs, six to seven sections
instead of two, the four categories collapsed into "solid" and "shaky", and 750–800 words
of re-narrated prose. All three also invented arguments the source never made, each a
different set — the signature of nothing binding the output.

The failure is wrong-*shaped* output, not a rule being knowingly broken. For that class,
a positive output contract binds and a prohibition does not: an agent under a competing
incentive negotiates with "don't do X", but either matches a stated shape or doesn't. So
`SKILL.md` specifies the shape — column set, type tests, a line grammar for the
composition — rather than listing what to avoid.

This was not theoretical. One prohibition survived into the first draft ("no closing
paragraph"). It leaked in two of three runs, with agents appending exactly the prose
analysis it banned. Replacing it with a bounded, ID-only `RESTS ON:` line *defined as the
last line of the output* fixed it. The impulse needed somewhere to go, not a stronger ban.

Four further defects were found and fixed across later rounds: the constraint test
misfiring on scale measurements, undefined uncertainty propagation through `gates`, an
unpinned `gates` grammar, and the load-bearing flag conflating "would change the decision"
with "currently holds it up".

Full testing record is in the pull request that added this skill.
