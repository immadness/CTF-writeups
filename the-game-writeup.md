# TryHackMe — The Game

**Category:** Reverse Engineering / Game Hacking  
**Platform:** TryHackMe  
**Flag:** `THM{I_CAN_READ_IT_ALL}`

---

## Challenge Overview

We're given a zip file containing a Windows executable: `Tetrix.exe` — a Tetris clone. The objective is to get a flag, which the game supposedly reveals after scoring **9999 points**. Anyone who has played Tetris knows that's not a casual afternoon.

---

## Initial Recon

Unzipping the archive gives us the binary:

```bash
unzip challenge.zip
# extracts: Tetrix.exe
```

Since we're on Linux, we need Wine to run it:

```bash
sudo pacman -S wine   # or apt/yum equivalent
wine Tetrix.exe
```

The game launches and plays like a standard Tetris. The score counter is visible on screen. The challenge is clear — reach 9999 to unlock the flag.

---

## Approach 1: Memory Manipulation with scanmem

The intended path (or at least the fun path) is to cheat the score by editing the game's memory at runtime. `scanmem` lets you scan a running process's memory for specific values and modify them.

```bash
sudo pacman -S scanmem
sudo scanmem
```

With the game running under Wine, we attach scanmem to the Wine process and start hunting for the score variable.

### Attempt 1 — Search for 9999 directly

With score at 0, we searched for the target value `9999`:

```
> pid <wine_pid>
> 9999
```

This returned **79 results** — still too many to blindly set them all without risking crashing the game.

### Attempt 2 — Score some points and narrow it down

The correct approach is to:
1. Score some points (e.g., 100)
2. Search for that value to find candidates
3. Score more points, search again
4. Repeat until one address remains, then set it to 9999

We searched for `100` after scoring 100 points:

```
> 100
```

Result: **192,600 matches** — the score value is somewhere in that haystack, but filtering it down requires playing multiple rounds of Tetris to progressively eliminate false positives.

That requires patience. We didn't have patience.

---

## Approach 2: strings

Before going further down the memory rabbit hole, it's worth asking a simpler question: is the flag even hidden, or is it just sitting in the binary as plaintext?

```bash
strings Tetrix.exe
```

Buried in the output:

```
THM{I_CAN_READ_IT_ALL}
```

There it is. The flag was stored as a plaintext string inside the executable — no scoring required.

---

## Flag

```
THM{I_CAN_READ_IT_ALL}
```

---

## Takeaways

- Always run `strings` on a binary before anything else. It costs nothing and occasionally hands you the answer immediately.
- `scanmem` is a legitimate approach for runtime memory patching, but it requires iterative narrowing — scan, change state, scan again.
- The flag name is self-aware: `I_CAN_READ_IT_ALL` is a direct nod to `strings` reading everything out of the binary.
