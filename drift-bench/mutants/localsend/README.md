# LocalSend mutants

The drifts of section 3 of the protocol (`paper/protocol.md`), for LocalSend v1.19.0 at commit 44f48f18. Each file is one patch. Its first line says what it changes, and `git apply <file>` applies it to the app at that commit. They were written after the traces were frozen (commit 3a700fc of mobile-agent-testers, branch `eval-localsend-1-traces`) and are committed here before any replay on a mutant. No proposer was run while they were written.

## How they were chosen

The rule of the protocol: for each class, take the screens in decreasing order of the number of kept traces that visit them, and write one change of that class on each, until the class has 5 mutants.

Visits by the 7 kept traces:

| Screen | Kept traces that visit it |
|---|---|
| R: Receive tab (home page, with the navigation) | 7 |
| S: Send tab | 7 |
| M: message dialog (Text button) | 7 |
| T: Settings tab | 7 |
| A: address dialog (Manual sending) | 6 |
| Troubleshoot page, color dialog, language dialog | 1 each |

Ties are broken by the order of the first visit in the traces: R, S, M, T, A. So each class has one mutant on each of these 5 screens.

- D2 (identifier) does not apply on LocalSend: no step of a kept trace targets an identifier (168 taps by text, 3 by position, 0 by id).
- D5 delays cycle through the three values of the protocol in screen order: R 1 s, S 5 s, M 15 s, T 1 s, A 5 s.
- D1 adds the suffix ` ›` to a text, in every language, so the change is the same for every persona.
- D4 dialogs and D5 waits are written in Dart in the file that shows the screen. The app's strings come from committed generated files, so a mutant changes widget code, not the translation files.
- D6 is the control: any green D6 is a false green.

On T, the only element a kept trace taps is the Language row (ravi-212224, step s16). The D1 and D3 mutants on T therefore change that row, and only that trace can see them. The other traces see T only through the check on its title.

## The mutants

"Seen by" counts the kept traces with a step or a check that the change touches, before any step where the trace stops.

| Mutant | Class | Screen | Seen by |
|---|---|---|---|
| d1-receive-send-tab | D1 | R | 7 |
| d1-send-text-button | D1 | S | 7 |
| d1-message-confirm | D1 | M | 7 |
| d1-settings-language | D1 | T | 1 (ravi-212224) |
| d1-address-confirm | D1 | A | 4 (not lucia, who cancel) |
| d3-receive-tab-order | D3 | R | 7 |
| d3-send-text-last | D3 | S | 7 |
| d3-message-buttons-swapped | D3 | M | 7 |
| d3-settings-language-first | D3 | T | 1 (ravi-212224) |
| d3-address-buttons-swapped | D3 | A | 6 |
| d4-receive-start-dialog | D4 | R | 7 |
| d4-send-tab-dialog | D4 | S | 7 |
| d4-message-dialog-before | D4 | M | 7 |
| d4-settings-tab-dialog | D4 | T | 7 |
| d4-address-dialog-before | D4 | A | 6 |
| d5-receive-nav-1s | D5 | R | 7 |
| d5-send-text-button-5s | D5 | S | 7 |
| d5-message-confirm-15s | D5 | M | 7 |
| d5-settings-content-1s | D5 | T | 7 |
| d5-address-confirm-5s | D5 | A | 4 (not lucia) |
| d6-receive-no-link-button | D6 | R | 7 |
| d6-send-no-selection-card | D6 | S | 7 |
| d6-message-dialog-gone | D6 | M | 7 |
| d6-settings-tab-ignored | D6 | T | 7 |
| d6-address-dialog-gone | D6 | A | 6 |

On the unchanged app, ravi-212224 passed 17 of 20 replays: three failed the check after step s15 (Back). A failure of that trace at s15 on a mutant is reported with this note.

## Building a mutant

The same recipe as the unchanged app (Flutter 3.41.9, pinned by the app's `.fvmrc`), plus the patch, from the app's folder:

    git -C .. apply <mutant>.patch
    flutter pub get
    flutter build apk --debug --target-platform android-x64

The SHA-256 of each mutant APK is recorded in the result file.

Build control: before the mutants, the unchanged app is built once with the same command and no patch, and its 7 traces are replayed 3 times. This checks that a build in the job, instead of the published APK, does not change the replays by itself.
