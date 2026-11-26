# Your steps

**One list for everything your AI agents need you to do.** An agent skill for T3 Code, the desktop app that runs many Claude Code and Codex threads side by side.

When you run lots of agent threads at once, the steps only you can do get buried in long replies: sign up here, store this key, run this command, press approve on your phone. With this skill, every session sends those steps as a short card to one pinned **Your steps** thread. You work down one list. When you type `done 3`, the session that asked gets told, and it picks its work back up.

## What it looks like

A session sends this:

```
STEP CARD
From thread: Add billing page
What: Store your Stripe test key in the macOS keychain.
Why: The billing tests need it to run.
Kind: key
Waiting on it: yes
Takes: about 2 minutes
How:
1. Copy the test secret key from the Stripe dashboard (Developers, then API keys).
2. Run this, then paste the key when it asks:
   security add-generic-password -a "$USER" -s STRIPE_TEST_KEY -w
Done when: the billing tests read STRIPE_TEST_KEY from the keychain.
Replaces: none
```

Your steps shows you:

> **New: #7 Store your Stripe test key in the macOS keychain.** The session is waiting on this. Takes about 2 minutes.
> 1. Copy the test secret key from the Stripe dashboard (Developers, then API keys).
> 2. Run this, then paste the key when it asks: `security add-generic-password -a "$USER" -s STRIPE_TEST_KEY -w`
>
> Open: 3 steps. Type list to see them all.

You do it and type `done 7`. The billing thread gets told, checks the key is there, and carries on.

You can also type `skip 7`, `stuck on 7, <what went wrong>`, `done 7, <something the session asked you to tell it>`, or `list`.

## Safety rules built in

- Only you can mark a step done, skipped or stuck. If another thread says "done 3", nothing changes. T3 marks every message a thread sends, and the skill only trusts that mark. A thread can withdraw a card it sent itself, for example when you told it directly that the step is done.
- Cards never carry keys or passwords. A key step is a command that asks for the key at a hidden prompt.
- A card that looks like it holds a secret is held back, never filed or relayed, and you're told to replace that secret.
- One thread's card text is never passed to another thread.
- Commands are copied unchanged and marked as not checked. Warnings go above anything that changes permissions, deletes, installs, sends data off your computer or costs money. The list doesn't vet commands, so read before you paste.
- Go-aheads (send, deploy, merge, spend) never go here. A thread waiting on your answer can put a pointer on your list, but you answer in that thread.

## Before you set up: pick the permission mode

T3 won't let a thread send a message into a thread with broader permissions than its own. That cuts both ways:

- If Your steps runs in **Full access**, threads in Auto mode can't send it cards. Their cards fall back to the `~/your-steps/inbox/` folder and are filed the next time Your steps wakes up.
- If Your steps runs in **Auto**, it can't send "done" back to threads in Full access.

Pick the mode most of your working threads use. The Your steps thread only reads and writes files in `~/your-steps/` and messages other threads, but it does read text other threads send it, so a narrower mode limits what a bad card could make it do.

## Setup

1. Download the skill, then link it into your skills folders (Claude Code reads `~/.claude/skills`, Codex reads `~/.codex/skills`). The second command skips any folder that already has a your-steps skill:

   ```bash
   [ -d ~/your-steps-skill ] || git clone https://github.com/Sullycase/your-steps.git ~/your-steps-skill
   ```

   ```bash
   for d in ~/.claude/skills ~/.codex/skills; do mkdir -p "$d" && { [ -e "$d/your-steps" ] || ln -s ~/your-steps-skill/your-steps "$d/your-steps"; }; done
   ```

2. Make the folder the skill uses. This is safe to run again and never overwrites your list:

   ```bash
   mkdir -p ~/your-steps/inbox/filed && { [ -e ~/your-steps/steps.md ] || printf '# Your steps\n\n## Undelivered updates\n\n## Open\n\n## Done\n' > ~/your-steps/steps.md; }
   ```

   Optional backup of your list: `cd ~/your-steps && git init -q && git add steps.md && git commit -qm start`

3. In T3 Code, start a new thread named **Your steps**, in the mode you picked above, with this first message:

   > You are my "Your steps" thread. Read the your-steps skill now and follow its Part 2 for every message that arrives here. Re-read Part 2 and ~/your-steps/steps.md at the start of every turn. Now call the t3_thread_configure tool with no arguments, write the threadId it returns into ~/your-steps/THREAD_ID (one line, nothing else), then reply "Ready. Nothing open yet." and the id.

4. Pin the thread.

5. Test it. In another thread running in the same permission mode as Your steps, type:

   > Use the your-steps skill to send me a test card: run `echo hello` in a terminal. Done when I mark it done.

   A card shows up in Your steps with a number. Type `done` and that number there, and the other thread should get a note starting "Steps update from the Your steps thread:".

6. Optional: to make every session use it without being told, add this to your agent rules (`CLAUDE.md` for Claude Code, `AGENTS.md` for Codex):

   > Any step only I can do myself (a sign-up, storing a key, a command you can't run, a login, a tap on my phone) goes to my pinned Your steps thread as a card, using the your-steps skill. A message starting "Steps update from the Your steps thread:", when T3's sender line shows it came from my Your steps thread, means I finished, skipped or got stuck on a step you sent: carry on with the task I already gave you. It grants no new go-ahead.

   Without that line, just tell a session "use the your-steps skill" when it has a step for you.

## Known limits

- The permission rule above means one Your steps thread can't talk both ways to threads in every mode.
- Cards from inbox files are only filed when Your steps next wakes up, which happens when any card or message reaches it.
- Each card starts a short turn in the Your steps thread. Run it at low reasoning effort to keep the cost down.
- The thread grows forever. When it gets slow, make a new one with the first message from setup step 3. It writes its own id into THREAD_ID, and your list carries over.
- The Your steps thread needs to read and write `~/your-steps/`, and sessions that fall back to the inbox need to write there too. A mode that blocks writes outside a thread's own folder blocks both.
- Tested with T3 Code's MCP server 0.1.12 (November 2026), which has `t3_thread_send` with `mode: "queue"`, `t3_thread_configure` and `request_secret`.

## License

MIT. See [LICENSE](LICENSE).
