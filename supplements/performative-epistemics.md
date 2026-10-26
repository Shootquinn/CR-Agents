# Performative Epistemics

Performative epistemics is the pattern where a piece of writing narrates its own rigor instead of just being rigorous. It's a byproduct of RLHF-style training: reward models trained on human preference data learn that hedging, caveats, and visible self-correction score well, so the policy learns to perform those moves rather than let the underlying content carry them.

It shows up as a handful of recognizable tics. Self-narrating honesty: sentences whose only content is "we are being honest right now." Restated conclusions: a number or comparison given once, then repeated as if it were a separate insight. Throat-clearing headers: a framing clause before the actual claim, telling the reader how to feel about what's coming instead of just saying it. Meta-commentary aimed at the reader: staging an argument instead of making it. Table sandwiching: explaining what a table is about to show, then explaining what it just showed, padding actual data with narration on both sides. Changelog leakage: references to a prior draft's numbers instead of just stating the current one.

None of these add information. They add the appearance of a mind checking its own work, which is a different thing from a mind having actually checked its work. The tell is that you can delete the sentence and lose nothing but word count, versus a real caveat, which if deleted would let a false or unsupported claim stand unchallenged.

The fix isn't "add more hedging" or "remove all hedging," it's noticing which sentences are load-bearing and which are decorative, and only keeping the first kind. A document that states a number and immediately flags a real problem with that number is doing epistemics. A document that states a number and then tells you it's being brave for admitting the problem is performing it.

## Real examples

All pulled from the same document, before trimming.

- Self-narrating honesty: "That is where the position gets uncomfortable and we say so." / "The consequences are real and we state them rather than waiting to be told." / "that is an honest cost to our position."
- Restated conclusion: "The margin ... falls from a factor of 20 at our number to 1.89 at 0.530 and to 1.02 at 0.983, which is to say that at the heaviest fixed-site sizing case the three prospecting cells sit essentially ON that boundary" — the number was already stated; the clause after "which is to say" adds nothing.
- Throat-clearing header: "A6 (why we keep 0.05 anyway, and it is a sourced position rather than an assertion, though the margin is narrower than the last draft claimed)" — three separate defensive moves before the argument even starts.
- Meta-commentary staging the reader: "Push first on the duty cycle... Push second on the architecture, which a critic will likely go to first... Push third on the allowance, and this is a precise open question rather than a general one." / "Attack the duty cycle first."
- Table sandwiching plus self-congratulation: "That table is the single strongest support this claim has, it survived independent recomputation of every component mass, every row closure, and all six conversions, and it says what we have been saying: the sizing policy, not the hardware, sets the mass." — a full sentence praising the table that was just shown.
- Changelog leakage: "why we keep 0.05 anyway... though the margin is narrower than the last draft claimed" / "the strongest case against us, stated in full and now heavier than the last draft admitted" — draft history stated as if it were part of the claim itself.
