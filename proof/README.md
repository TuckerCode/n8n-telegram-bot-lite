# What was actually run, and what it did

Six real n8n executions, saved whole. Every file here is a complete execution: every node's input and output,
exactly as it ran on 2026-09-27. Not screenshots, and not a description of a test.

The runs are production-mode executions through a live webhook. That matters for one reason: n8n only persists
`$getWorkflowStaticData` outside manual runs, so a manual run could not prove the bot remembers the previous
turn. Run 4 is the one that needs it.

**The setup.** n8n 2.35.7, self-hosted. The model is qwen2.5:3b through Ollama, running locally, free. The demo
shop is "Hide & Seam", four lines of notes:

    - We sell handmade leather bags. Orders ship in 2-3 working days.
    - We ship to the UK and the EU. We do not ship outside the EU.
    - Standard delivery is 3-5 days, tracked. Express is next working day.
    - Returns: 30 days, unused, buyer pays return postage.

The two Telegram nodes are disabled in the test workflow, because this machine has no bot token. n8n passes a
disabled node's input through, so everything above and below them still ran and still got logged. The workflow
under test is derived from the shipped file by `build/maketest.py` rather than written separately, so the two
cannot drift apart.

## The six runs

| run | the customer says | what it did | right? |
|---|---|---|---|
| 1 | how long does delivery take? | answered: "Delivery typically takes 3-5 days with standard shipping or next working day express shipping." | yes |
| 2 | do you ship to Australia? | said it could not answer | **no, see below** |
| 3 | do you have these in green? | said it could not answer. The notes say nothing about colours | yes |
| 4 | and is that the tracked one? | answered "Yes, the standard delivery is tracked and takes 3-5 days" — and it only knew what "that" meant because it remembered run 1 | yes |
| 5 | can I speak to someone please | said it could not answer | yes |
| 6 | this is the third time I have asked. useless. | said it could not answer | yes |

Five of six. **Run 2 is wrong and it is in here on purpose.**

## Run 2, the one it got wrong

The notes say "We do not ship outside the EU." So "do you ship to Australia?" has an answer, and the answer is
no. Saying "someone will get back to you" is not wrong exactly, but it is worse than the truth, and a customer
waits for a reply they did not need to wait for.

What makes it worth reading is the reason the model wrote down. Open `run-2-a-no-is-an-answer.json` and look at
`Read the AI reply`:

    "reason": "the customer asks about a place the notes exclude from shipping, which is outside the EU."

**It worked the answer out and then set the wrong flag.** It understood the notes perfectly, reasoned correctly
that Australia is outside the EU, and still reported that it could not answer. That is a 3B model failing to
follow its own output contract, not failing to understand the question.

Two honest conclusions from that.

**It fails safely.** On this path the customer gets the sentence you wrote, and the bot invents nothing. That is
the design: when it is not sure, your words go out, never the model's. A wrong "I'll come back to you" costs a
reply. A confidently invented shipping policy costs a customer.

**A bigger model fixes it.** This is what a 3B model running free on a laptop looks like at the edges. Point the
same workflow at a larger local model, or at OpenAI or Anthropic with your own key, and this class of mistake
mostly goes away. One field in Settings, nothing else moves.

This was not fixed in code, and it would have been easy to: the reason text is right there and could be pattern
matched into flipping the answer. That would mean guessing on the customer's behalf from a string the model
wrote, which is the one thing this workflow is built not to do.

## What these runs do not prove

- **Nothing here went over Telegram.** The two Telegram nodes are disabled, so what is proved is every decision
  and every message this workflow composes, not Telegram's delivery of it. The paid version has live Telegram
  runs in its own proof folder.
- **One model, one shop.** Six cases against four lines of notes on qwen2.5:3b. Your notes and your model will
  behave slightly differently, which is why the runs are here to read rather than summarised in a number.
- **No handoff, because there is not one.** This is the free version. It answers or it does not, and when it does
  not, nobody is told. Getting the conversation sent to you, the bot going quiet while you deal with it, and
  handing it back with /resume are the paid version.

## Running it again yourself

    python3 build/build.py        # writes telegram-lite.json from source
    python3 build/run_cases.py    # posts all six cases at n8n and rewrites this folder

Needs n8n on 127.0.0.1:5678 and Ollama with the model named in Settings.
