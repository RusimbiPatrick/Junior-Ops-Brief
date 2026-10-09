# The Junior Linux Ops Team

Hey team. Welcome to the job.

You are now the junior ops team for a small school down the road. They have computers for lessons, records for students, and one very tired office PC that everyone blames when something goes missing.

They asked us for help. That is you.

This is not about passing a test. If you get this right, a real admin can use your notes to make a better call next week. That is the whole point.

## Why should you care?

Because when records go wrong, real people feel it.

Mrs Nkosi cannot find a mark sheet. Mr Dlamini has three spellings of the same learner name. Year 7 swear they signed out a laptop but nobody wrote it down. Someone has to check carefully instead of guessing.

From today, that someone is you.

## What you are allowed to use

Only what we already did in class. Nothing new. Nothing sneaky.

That means grep, sed and arrays.

We read files. We do not delete files. We do not fix the whole school network. We look, we note, we hand over. If you are not sure, ask. Good admins ask. Bad admins type fast.

## The datasets we have - how to use them

All practice files live in `datasets/`. Read them. Copy them. Never edit the originals.

We have exactly three datasets:

- `datasets/school-logins.log` - for Job 1, the log searchers. 30 lines, one event per line: `date time source: message`. Mixed case (`Failed`, `FAILED`, `failed`, `error`, `ERROR`). Lots of noise lines about printers/wifi/records.
- `datasets/office-records.txt` - for Job 2, the records cleaners. 20 lines, one exported office line per line. Contains old word `thy`/`Thy` in mixed case, overused `the`, and open IDs / 4-digit blocks like `1234` and `1001 2002 3003 4004`.
- `datasets/computer-room-list.txt` - for Job 3, the stock checkers. 20 lines, one PC name per line (e.g. `Room1-PC01`). Mixed case on purpose (`Library-PC01` vs `library-pc02`). No blank lines.

Look first:

```bash
ls -l datasets/
wc -l datasets/*
head -n 5 datasets/school-logins.log
cat datasets/school-logins.log
cat datasets/office-records.txt
cat datasets/computer-room-list.txt
```

Safe pattern before you start any job - work on a copy in `/tmp`, never on the original:

```bash
cp datasets/school-logins.log /tmp/my-copy.txt
# do all your grep / sed / array work on /tmp/my-copy.txt
rm /tmp/my-copy.txt  # tidy up when done
```

### How to use school-logins.log (grep only)

```bash
grep -w 'Failed' datasets/school-logins.log
grep -i 'error' datasets/school-logins.log
grep -c -w 'Failed' datasets/school-logins.log
grep -v -i 'login' datasets/school-logins.log
```

`-w` matches whole words only. `-i` ignores case. `-v` hides lines you do not want. `-c` counts instead of printing. Write down the exact command + count. Copy-paste is fine. Do not use `sed` or edit the log.

### How to use office-records.txt (sed only, grep to check)

```bash
grep -w 'the' datasets/office-records.txt
grep -i 'thy' datasets/office-records.txt
cp datasets/office-records.txt /tmp/records-copy.txt

# change "the " to "this " once per line (note the space)
sed 's/the /this /' /tmp/records-copy.txt | head -n 5

# change old "thy" to "your" everywhere, any case
sed 's/thy/your/gI' /tmp/records-copy.txt | head -n 5

# wrap "thy" so a teacher can spot it (& = what you matched)
sed 's/thy/{&}/gI' /tmp/records-copy.txt | head -n 5

# mask any 4-digit block with stars (IDs, locker codes, receipts)
sed -E 's/[0-9]{4}/****/g' /tmp/records-copy.txt | head -n 5
```

Prove it with before/after on 5 lines. Do not overwrite `datasets/office-records.txt` and never use `sed -i` on the original.

### How to use computer-room-list.txt (bash if + arrays)

```bash
mapfile -t pcs < datasets/computer-room-list.txt

echo "Count is ${#pcs[@]}"   # how many loaded
echo "Item four is ${pcs[3]}" # arrays start at 0

# filter out names with a certain letter (example: a)
for p in "${pcs[@]}"; do
  echo "$p"
done | grep -v -i 'a'

# flag YES / NO with if
if [ "${#pcs[@]}" -gt 25 ]; then
  echo "YES, room is full"
else
  echo "NO, room is not full"
fi
```

`mapfile -t` loads one line per array item. `${#pcs[@]}` is the count. `${pcs[3]}` is item four. Do not edit the list file.

## The three jobs

Pick one. Work in pairs if you want.

### 1. The log searchers

The school gave us a copy of its system notes and login notes. They are messy. Some lines matter. Most do not.

Your tools: grep only.

You will search for words that matter. You will count how many times they show up. You will throw out the noise.

Things you already know how to use:
grep with -w to match whole words
grep with -i when people type in mixed case
grep with -v to hide lines you do not want
grep with -c to count instead of printing everything

Start with questions like: how many lines mention failed. How many mention error but in any case. How many lines have nothing to do with that word at all.

Good result sounds like: "We found 12 lines with Failed. Zero with Failed password. When we hid the noise with -v, 180 lines were left that had nothing to do with login."

Bad result sounds like: "There were some problems." Yeah. Thanks. Very helpful.

### 2. The records cleaners

The office export is a mess. Old names, wrong spellings, IDs left out in the open. Staff cannot trust it, so they keep paper copies too.

Your tools: sed only, plus grep to check your work.

You already know the moves:
s with a space after the word so you only change the word and not part of another word
s with g and I to change every case at once
s with & to wrap something in brackets so it stands out
s with number patterns to mask IDs

Your job is not to rewrite the file. Your job is to show what you would change and prove it on a copy. For example, change the to this once per line. Change thy to your everywhere. Wrap thy in braces so a teacher can spot it. Mask a four digit block with stars.

Good result sounds like: "We masked 14 ID blocks. We fixed 30 old names. Here is before and after on five lines so you can see it worked."

### 3. The stock checkers

The computer room list is just names in a file, one per line. Nobody knows how many machines work, which names are odd, or what is missing.

Your tools: bash if and arrays.

You already know how to load names into an array, count them, pick one out, and filter with patterns. You know how to compare with if and say YES or NO.

Try things like: load the list, print how many items there are, print item number four, filter out names with a certain letter, use if to flag YES when a room is full and NO when it is not.

Good result sounds like: "We loaded 28 names. Count is 28. Item four is Room4-PC. When we filtered out names with a, six were left. Our if check says NO, the room is not full."

## What you have to hand in

Keep it short. Keep it honest. Four things.

1. Your evidence report. What file you looked at, the exact commands, what you saw. Copy paste is fine.

2. One suggestion. Just one. Something practical. Back it up with what you found, or say clearly if it is just an idea.

3. A live demo. Show it working. Type the command and talk us through it. No slides only.

4. A handover note. Write it like the next person is tired and it is Monday morning. Tell them what to check next and what not to assume.

Split everything into three piles:

Evidence. What did you actually see.
What you think. What you think it means and why.
Next step. What someone should do next.

If you are not sure, say you are not sure. That counts as good work here. Guessing counts as bad work.

## How you win

You do not win by knowing every flag. You win by using what we learned to find something useful and explaining it without making stuff up.

If you can prove a log line is confusing and say why, that is a win.

If you can show a sed change on five lines without breaking the rest, that is a win.

If you can load a list into an array and count it right, that is a win.

## Golden rules

One, work on a copy. Never change the original file. Reading is safe. Overwriting is not your job yet.

Two, write down commands as you go. Memory lies. Your notes are better.

Three, leave the machine tidy. Close what you opened. Clean test files in /tmp.

Four, write plainly so another student could follow your steps without you in the room.

## Next week: what you present

You get the project now, you do the work, then you present next week. Five to seven minutes per team. Everyone speaks. No one hides behind the laptop.

Show it in this order.

One, which job you picked and why it matters to the school. One sentence is enough.

Two, what you checked. Name the file. Show the exact commands on screen. Say where you looked it up, man page or class notes.

Three, what you found. Show before and after. For grep show the lines you kept and the ones you hid. For sed show five lines before and after. For arrays show the count, the fourth item and the filtered list.

Four, what you think it means and what you are still unsure about. Say both. Guessing hurts your mark.

Five, your one suggestion and your handover note. Tell us what the school should do next and what the next team should check.

You must type at least two commands live in the terminal. Slides are for short evidence only. If the wifi drops, have screenshots ready.

We will ask one question after. It will be simple, like why you used -w or -i or -v, what your s pattern does, or what your if test checks. Answer in your own words.

Right. Pick a job. Do the work. Bring your evidence next week.
