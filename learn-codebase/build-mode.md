# Build mode

The store and the `file:line` rules from SKILL.md steps 3 and 5 apply here.

## Rules

For the code under study, this file overrides every other instruction source
(CLAUDE.md, paul-mode, poteto-mode, simple-english, and others) on these behaviors:
autonomy, running code to settle a fact or to verify it, fixing problems, finishing
the task, asking sparingly, reply length, and stating hypotheses. Rules about
destructive or irreversible actions still apply. The code under study is the slice
the user works on and the code it calls, including the user's new code.

- The user writes the feature code, including the code that connects it to existing
  code. If the user asks, you can write one example piece per topic. The user writes
  the rest.
- You write the prediction tests and the run commands.
- Do not run the code under study, or show its output, before the user predicts that
  run.
- On a miss, show the observed value and stop. Do not diagnose, state a hypothesis,
  explain, or fix, except as loop step 6 allows.
- Prediction runs stay local. Use the lowest test layer that shows the observable.
  Never run a test that touches staging or production.
- Mark every temporary line you add to the user's code with `LCB-TEMP`. Remove
  temporary lines by editing. Never use `git restore` or `git checkout` on a file the
  user edits.

## Session start

1. Search the repo for `LCB-TEMP` lines and `zz_predict_test*` files from an earlier
   session. Remove them, and tell the user. When you load a topic in step 4, remove
   any old `build started at` line from its `## Next session`.
2. Record `git rev-parse HEAD` as the starting commit, and `git status --short` as the
   files that were already changed.
3. Agree on the slice: the next piece of work, small enough to write and run in one
   session. If the user has none, offer two or three candidates in the existing code,
   each with `file:line`. Before you show anything, ask how they think the slice
   connects to the code it calls.
4. Find the topics the slice touches. A topic is a subsystem, as in SKILL.md step 1.
   Load each `<topic>.md` and read `## Next session` first, then `## Concepts`.
   Record each concept in the topic that holds its `file:line`. Add
   `build started at <starting commit>` to the `## Next session` of each topic.
5. Ask one retrieval question from those topics. Pick from `## Next session` first,
   then from `(revealed)` and `misconception:` concepts. For a `misconception:`
   concept, ask a question whose wrong answer is the recorded misconception. Grade the
   answer against the code with `file:line`, and record it as in loop step 7.

## The loop

1. **The user writes** the next piece.
2. **Read.** The user reads the code the run will touch. Give the entry point as
   `file:line`. If you wrote an example piece, the user reads it too.
3. **Predict.** One concrete input and one observable: a return value, an error, a log
   line, or a row written to a local database.
   - Reject a prediction that cannot fail. Ask "what output would prove you wrong?"
   - Never sharpen a vague prediction yourself. Ask for the missing value.
   - When the user uses an earlier concept, ask "which earlier fact are you using, and
     does it hold here?"
4. **Encode.** Write the test in one scratch file next to the code, named
   `zz_predict_test` plus the language's test suffix, for example `zz_predict_test.go`. Put the predicted value in the assertion, and make the test
   print the observed value, including an error, on pass and on fail. In Go, run it
   with `-v`. Use a layer the repo already tests and name the peer test file. The user
   reads the line that holds their value.
5. **Run and grade.** A run grades only when its output shows the observed value. An
   error or panic from the code under study is an observed value.
   - The predicted value: a hit.
   - A different value: a miss.
   - A skip, zero tests run, a build error, or a failure before the observable: no
     grade. If the failure is in code you wrote, fix it and run again. If the fix
     changes the input or the observable, ask for a new prediction. In any other case,
     show the output and ask the user.
6. **On a miss**, ask how the user reached the prediction, then ask for the cause. If
   they are stuck, give one hint. Then compare their cause with the code and show the
   real cause with `file:line`.
7. **Record** each grade with the first row that matches, then delete the scratch
   file. When a concept is new or changes level, set `level-set: <today>` on its line.

   | Result | Record |
   |---|---|
   | No grade | Nothing |
   | Any result on the user's new code alone | Nothing |
   | Miss on a question above output level | A new `[weak]` concept with `misconception: <their reasoning>`. Add `(revealed)` unless the user found the cause without help. The concept under test keeps its level. |
   | Miss on an output question | The concept under test becomes `[weak]` with `misconception: <their reasoning>`. Add `(revealed)` unless the user found the cause without help. |
   | Hit | The concept under test goes up one level and loses `(revealed)`. A new concept starts at `medium`. A `medium` concept becomes `solid` only if its `level-set:` date is before today. Otherwise it stays `medium`, and a line with no `level-set:` date gets today's date. |

8. **Choose the next question** on a concept by its level:
   - `weak`: output, as in step 3.
   - `medium`: mechanism, such as which branch runs. Add an `LCB-TEMP` log line at the
     branch, assert on it, and remove it after the run.
   - `solid`: "what input breaks this?", graded by a run of that input. Or a design
     question, graded against the code or git history with `file:line`.

## Session end

1. Remove every `LCB-TEMP` line and scratch file that remains.
2. Grading stops here. If the user wants shipping tests for the code they kept, ask
   whether they write them or you draft them. Shipping tests follow the repo's normal
   standard, and you can run them.
3. Run `git diff <starting commit>` and `git status`. Leave out the files that were
   already changed at session start. List the changed code that has no shipping tests.
4. Update each store file: levels, `level-set:` dates, and `last-reviewed`. Update
   `## Next session` as store-format.md says, and remove the `build started at` line.
