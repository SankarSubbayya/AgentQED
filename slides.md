<!--
Render with Marp:
  npx @marp-team/marp-cli slides.md -o slides.pdf
  npx @marp-team/marp-cli slides.md -o slides.pptx
Live preview in VSCode: install the "Marp for VS Code" extension.

[YOUR NOTE] markers flag spots where your own voice will read more authentic
than mine. Rewrite those in your own words before presenting.
-->

---
marp: true
paginate: true
theme: default
style: |
  section {
    background: #0d0b14;
    color: #ece9f4;
    font-family: -apple-system, "Segoe UI", system-ui, sans-serif;
    padding: 56px 70px;
    font-size: 22px;
  }
  h1, h2 { color: #c4b5fd; font-weight: 600; letter-spacing: -0.02em; margin-bottom: 0.4em; }
  h1 { font-size: 2.3em; }
  h2 { font-size: 1.55em; }
  h3 { color: #a78bfa; font-weight: 500; }
  strong { color: #a78bfa; }
  code { background: #1c1825; padding: 2px 6px; border-radius: 4px; color: #ddd6fe; font-size: 0.85em; }
  pre code { display: block; padding: 14px 18px; line-height: 1.45; font-size: 0.78em; }
  blockquote { border-left: 3px solid #7c3aed; color: #a1a1aa; padding-left: 18px; font-style: normal; }
  a { color: #a78bfa; text-decoration: none; border-bottom: 1px dotted #7c3aed; }
  table { border-collapse: collapse; font-size: 0.85em; }
  th, td { border: 1px solid #2a2434; padding: 6px 12px; text-align: left; }
  th { background: #1c1825; color: #c4b5fd; }
  ul { line-height: 1.6; }
  .small { font-size: 0.8em; color: #a1a1aa; }
---

# AgentQED

### Write a math proof in English. A theorem prover decides if it's actually right.

<br>

[YOUR NOTE: your name, team, date]

<br>

<span class="small">proof-agent.vercel.app · github.com/SankarSubbayya/AgentQED</span>

---

## The problem

A natural-language proof can be wrong in ways that read fluent.

A skipped case. A quantifier flipped. An induction step that quietly assumes what it set out to prove.

There is no "is it right" button. You ask a friend. You ask a TA. You ask a language model, which will agree with you whether you're right or not.

Mathematicians solve this by writing proofs in a formal language a compiler can check. The most credible one right now is **Lean 4**, used at Imperial, CMU, and inside the project that formalized the polynomial Freiman-Ruzsa conjecture.

But learning Lean takes weeks. We wanted you to skip that part.

---

## What AgentQED does, in one diagram

```
  your proof (English, voice, photo, PDF)
            │
            ▼
  Gemini 3.1 Pro  ─────────►  Lean 4 source
            ▲                       │
            │                       ▼
   read errors, edit       Vercel Sandbox (Firecracker microVM)
            │                       │
            └──── retry  ◄──── lean compiler verdict
                                    │
                                    ▼
                            verified | rejected
```

The agent loop runs up to **12 retries**. On easy proofs it finishes in one. On harder ones it needs three or four edits, each driven by the actual compiler output, not by guessing.

---

## A real example

You type:

> Prove that for all natural numbers $n$, $0 + 1 + \dots + n = \dfrac{n(n+1)}{2}$.

Gemini writes this:

```lean
def sumTo : Nat → Nat
  | 0     => 0
  | n + 1 => (n + 1) + sumTo n

theorem sum_formula (n : Nat) : 2 * sumTo n = n * (n + 1) := by
  induction n with
  | zero => rfl
  | succ k ih =>
    simp [sumTo, Nat.mul_add, Nat.add_mul]
    omega
```

The sandbox runs `lean Proof.lean`. It compiles. The agent stops.

You did not need to know what `omega` does to get here. (It dispatches linear arithmetic.)

---

## Why not just have the LLM grade the proof

Because language models are confident liars about math.

A model can tell you your proof of Fermat's Last Theorem is correct. It will say that in a complete sentence. It will not stop saying that until you stop asking.

The Lean compiler does not have a personality. If your proof has a gap, the term it expects to typecheck does not typecheck, and you get an error pointing at the line. That error is what the agent reads to figure out what to fix.

Compiler-verified, not vibes-verified.

---

## Things you can throw at it

| Input | What happens |
|---|---|
| Plain text | Straight to Gemini |
| LaTeX | Same path, math notation preserved |
| Photo of handwriting | Vision pass first, then translation |
| Multi-page PDF | Page-by-page extraction, then a single Lean file |
| Microphone | Web Speech API streams a transcript while you talk |

The voice + photo paths exist because the people we tested with were mathematicians, and mathematicians work on paper.

---

## What you get back, and in what order

We deliberately put the **Key Insight** at the top of the result, not the code. One sentence about the idea that makes the proof work, with the relevant identity rendered as math. Below that, the **proof structure** as an indented tree:

```
Induction on n
├── Base: n = 0, both sides are 0
└── Step: assume true for k, prove for k+1
    ├── expand sumTo (k+1)
    ├── distribute 2 *
    └── apply ih, finish with omega
```

The full Lean code is collapsed by default. Most people want to understand the proof, not read syntax.

[YOUR NOTE: replace this with a screenshot of an actual verified result on the right-hand side. It sells the slide.]

---

## Lean potholes we fell into

Three things bit us, in order:

1. **`omega` does not do nonlinear.** It happily solves $2n + 3 = m$, gives up on $n \cdot k = m \cdot n$. We had to teach Gemini to reach for `ring` and manual `calc` blocks for anything with a variable multiplication.

2. **`/` on `Nat` is integer division.** The statement $\text{sum} = n(n+1)/2$ is meaningless in Lean's natural numbers when $n(n+1)$ is odd. We rewrite every sum formula as $2 \cdot \text{sum} = n(n+1)$ in the system prompt.

3. **Mathlib is huge and the sandbox doesn't have it.** We import only from core Lean, so the agent cannot rely on `Mathlib.Tactic.Ring`. Several proofs that look one-line in textbooks need a longer formal version.

[YOUR NOTE: keep the one of these that surprised you most. Cut the others. Three feels like a list; one feels like a story.]

---

## The agent loop, less abstractly

```ts
for (let attempt = 0; attempt < 12; attempt++) {
  const lean = await gemini.generate(systemPrompt, userProof, lastError)
  const result = await sandbox.run(`lean ${file}`, lean)
  if (result.status === "ok") return result
  lastError = result.stderr   // fed back into the next prompt
}
```

The interesting trick is what we feed back. Not the full stderr (Lean errors are noisy), and not just the first line (too little context). We strip the file path noise, keep the goal state at the failure point, and prepend the line of code that failed.

That single change took our success rate on the six sample proofs from about half to all six.

[YOUR NOTE: replace the number with whatever yours actually is. If you instrumented this, even better.]

---

## Why we picked the pieces we picked

**Gemini 3.1 Pro** for translation. It handles handwriting in photos better than the alternatives we tried, and it does not get stuck inside its own previous suggestion when you give it a compiler error.

**Vercel Sandbox** to run Lean. Each request gets a fresh Firecracker microVM. No shared filesystem, no cached state to corrupt the next user's run. Cold start is the real cost. About six seconds to spin up, instant for warm.

**Next.js + AI SDK** because the streaming UI matters. You see the agent talk to itself in real time, which makes the wait feel like progress rather than a loading spinner.

---

## Try it

<br>

# proof-agent.vercel.app

<br>

Six sample proofs live on the landing page. Modus ponens is the fastest. Sum of first n naturals is the most satisfying.

If you brought a proof on paper, take a photo and drop it in.

[YOUR NOTE: drop a QR code here. `qrencode -s 12 -o qr.png "https://proof-agent.vercel.app"` if you have qrencode, or paste a PNG from qr-code-generator.com.]

---

## What we'd do with another week

Mathlib in the sandbox, so the agent can lean on `ring` and `linarith` without us writing tactical workarounds.

A "show me the gap" mode, where instead of trying to fix the user's proof, the agent points at the specific step that does not formalize.

Sharing. Right now everything lives in `localStorage`, which means your verified proofs evaporate if you switch browsers. A simple persistence layer would let people send each other proofs the way they currently send each other graphs.

[YOUR NOTE: if there's a specific thing you wish you'd had time for, swap it in.]

---

## Built by

Sankar Subbayya · accurateai.org  
Ganesh Sankar · UC Berkeley

<br>

Code: github.com/SankarSubbayya/AgentQED  
Live: proof-agent.vercel.app · agentqed.pages.dev

<br>

Questions are welcome. So are counterexamples.
