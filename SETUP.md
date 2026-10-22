# Telegram AI bot, Lite — setup

About ten minutes. Three fields to fill in. No paid account, no API key unless you want one.

What you need:

- n8n, self-hosted. Built and tested on 2.35.7.
- A Telegram bot token, free, from @BotFather.
- An AI model. A free local one through Ollama is what this was built and tested on. OpenAI or Anthropic work
  too, with your own key.

---

## 1. Make the bot

Open Telegram and message @BotFather. Send `/newbot`, give it a name and a username. He replies with a token
that looks like `8012345678:AAE...`. Keep it.

## 2. Import the workflow

In n8n: Workflows, then Import from File, and pick `telegram-lite.json`. Nine nodes appear, plus three sticky notes explaining them — the notes are documentation on the canvas and do nothing when the workflow runs.

## 3. Add the Telegram credential

Click the node called **Send the answer**. Under Credential to connect with, choose Create new. Paste the token
from step 1. Save.

Both Telegram nodes use the same credential, so you only do this once. Click **Say you will come back to them**
and pick the credential you just made.

## 4. Fill in the Settings node

This is the only node you edit. Three fields matter:

**ai_model** — `qwen3:4b-instruct` if you are using Ollama and have pulled that model. Any model you have works. On
OpenAI or Anthropic, put their model name here instead.

**business_name** — what you are called. It goes in the prompt.

**knowledge** — the facts your bot is allowed to state, one per line, starting with a dash. This is the whole
brain. Anything not in here gets your cannot_answer_message instead of an answer.

Write it like this:

    - We sell handmade leather bags. Orders ship in 2-3 working days.
    - We ship to the UK and the EU. We do not ship outside the EU.
    - Standard delivery is 3-5 days, tracked. Express is next working day.
    - Returns: 30 days, unused, buyer pays return postage.

Short factual lines beat paragraphs. Write the four questions you answer most and stop.

The rest have working defaults:

| field | default | what it does |
|---|---|---|
| ai_provider | ollama | ollama, openai or anthropic |
| ollama_url | http://127.0.0.1:11434 | where your Ollama is |
| cannot_answer_message | "Thanks for your message. Someone will get back to you here shortly." | what the customer reads when the bot cannot answer. Your words, always. Change this to sound like you |
| history_turns | 10 | how much of the conversation the model sees |
| log_turns_kept | 200 | how much is kept per chat |
| prompt | the instructions | you can read it. Leave it alone until the rest works |

### Using OpenAI or Anthropic instead

**There is no API key in this workflow and there should not be.** n8n's creator hub forbids one, and the rule is
right: a template is shared, and a key typed into it is shared with it.

Ollama is the default and needs nothing — it is keyless and runs on your own machine.

For a paid provider, the key goes on the node instead of in the file:

1. Set `ai_provider` in **Settings** to `openai` or `anthropic`, and set `ai_model` to one of theirs.
2. Open the **Ask the AI** node.
3. Set **Authentication** to *Generic Credential Type*, then **Generic Auth Type** to *Header Auth*.
4. Create a credential and select it:
   - **OpenAI** — name `Authorization`, value `Bearer YOUR_KEY`
   - **Anthropic** — name `x-api-key`, value `YOUR_KEY`

n8n stores that credential encrypted and it never travels with the template, so you can export this workflow
and share it without leaking anything.

## 5. If you are using Ollama

Ollama has to be reachable from inside n8n. If n8n runs in Docker and Ollama runs on the host, then
`http://127.0.0.1:11434` is wrong from inside the container. Use `http://host.docker.internal:11434` on Mac and
Windows, or your host's LAN address on Linux.

Test it first, outside n8n:

    curl http://127.0.0.1:11434/api/tags

If that lists your models, Ollama is running. If it does not, start it and pull a model:

    ollama pull qwen3:4b-instruct

## 6. Turn it on and say hello

Save the workflow, then switch it Active, top right. Message your bot on Telegram. Ask it something your notes
cover. You should get an answer in a few seconds, longer on a slow local model.

Then open the Executions tab and read the run. Click **Read the AI reply** and you can see exactly what it
decided and why, in the `reason` field. That is the same thing you will look at any time it behaves oddly.

---

## What it does not do

Read this so nothing is a surprise.

- **It does not hand a conversation to a person.** When it cannot answer, the customer gets your
  cannot_answer_message and that is the end of it. Nobody is told, nothing is forwarded, and there is no way to
  take a chat over. That is the paid version.
- **Text messages only.** No voice notes, no photos, no documents.
- **It cannot message a customer first.** Telegram does not allow that, for anyone.
- **It only knows your notes.** It has no access to your shop, your orders, your stock or your CRM.
- **A small local model gets things wrong at the edges.** `proof/README.md` has a run where it worked out the
  right answer and then reported that it could not answer. It fails towards saying nothing rather than towards
  inventing something, which is the safe direction, but it does happen. A bigger model makes it rarer.

## When something is wrong

**No reply at all.** Is the workflow Active? An inactive workflow ignores Telegram entirely.

**"The requested webhook is not registered."** Same thing. Activate it.

**It says it cannot answer to everything.** Your knowledge field is probably empty or still has the placeholder
lines in it. Open Executions, click Read the AI reply, and look at `reason` — it says which rule stopped it.

**It answers with something that is not true.** Add the fact to knowledge. If it is inventing things that are in
no line of your notes, your model is too small. Try a bigger one.

**Ollama connection refused.** See step 5. Nearly always Docker networking.

---

If anything is wrong, email tucker@airankings.tech and tell me what happened.

The full version, with the handoff to a human, is at
whop.com/telegram-ai-bot-with-human-handoff
