# Example Use Cases

**1. Historical thinkers.** Compile the complete works of a thinker into a face. Ask it the
questions they never answered, on whichever model you chat with.

**2. Customer panel from transcripts.** Upload 20 interview transcripts as threads, one face
per participant, with `--face-speaker` naming the participant so only their turns are used. Run `face:diff` across the panel to see who is alike before reading a word, then
ask the whole panel the same question with `faces decide` and compare the numbers.

**3. A board of advisors.** Assemble a scientist, an artist, a pragmatist and a mystic. Route
every major decision through all four, and see the full range of consistent answers before
choosing a direction.

**4. The opposite view.** After building a face from your own writing (your `me` face), find
its furthest neighbor with `face:neighbors me --direction furthest`. That is the face least
like yours in the collection. Ask it the questions you asked yourself; the gap between the two
answers is where your blind spot is.

**5. A proxy for yourself.** Compile your own emails, notes, essays and voice memos into `me`.
Share it through a scoped API key so collaborators can consult your point of view when you are
unavailable.

**6. Drift over time.** Compile new material into a face every quarter, and compare it with
`face:diff` against a face compiled from the earlier material, to see whether a person's or an
organization's thinking is moving.

**7. Boolean composition.** You have a pragmatist and a visionary. Their union
(`@pragmatist | @visionary`) holds both outlooks. Their intersection (`@pragmatist & @visionary`)
keeps only the common ground. Their difference (`@visionary - @pragmatist`) is the visionary
without the practical instinct. Each is a face you can ask, and none needed another
document compiled.

**8. Synthetic focus group.** Build faces for an early adopter, a skeptic, a pragmatist, a
regulator and a loyalist. Run a product pitch or a policy change past each, with
`faces decide` for numbers and `faces choices` for the options each would weigh.

**9. Stress-testing an argument.** Compile a position into a face. Its nearest neighbors are
who already agrees; its furthest is who opposes it most. Ask the furthest for the sharpest
counter-argument.

**10. Your own judgement.** Compile your posts, threads and captions into a face, then
ask it what you would do: `faces choices @me:maestro-1 --state "<situation>" --question "<question>"`.
Compiling new material over time keeps it current.

**11. A deliberation pipeline.** Compile an `expert` face from domain literature. Make a
`critic` as `expert - consensus`, and a `synthesizer` as `expert | critic`. Ask `expert` first,
pass its answer to `critic`, then pass both to `synthesizer`. Each stage is one face; together
they are a pipeline built from set algebra.

**12. Debate between faces.** Ask one face, and hand its answer to another.
`faces chat:chat @penrose:maestro-1/gpt-5.4 -m 'What resolves the information paradox?'`, then
`faces chat:chat @hawking:maestro-1/gpt-5.4 -m 'Penrose holds: <his answer>. How do you answer him?'`.
Alternate for as many rounds as the exchange needs.

**13. Neutral analysis across faces.** Ask each face the same question with `faces choices`,
then give the options to a model with no face and ask where they converge. The model compares
both without taking either side.

**14. Steering your own pipeline.** Point an OpenAI- or Anthropic-compatible client at
`https://api.faces.sh/v1` and name the model as `@alias:maestro-1/<model>`: the face steers every
reply, and your code does not change.
