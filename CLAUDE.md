# StoryHeart

An audiobook player for Windows, made for keyboard and screen reader users (NVDA, JAWS, Narrator). C# and WPF on .NET 10. Owner: James Trimble. Hamster is a tester (she/her). It was called BookHeart until 0.2.0-beta.6, and AudiobookPlayer before that.

## Working rules

- Work on `main` and push to `main`. Don't create other branches or pull requests unless James asks.
- Commits are authored as James Trimble (the GitHub noreply address). Never add Claude attribution, `Co-Authored-By` or session trailers to commits.
- Don't build or release a new test version until James says go. Pushing to `main` is fine: it only runs the Build workflow.
- Never buy anything (domains, services) without a quote first and James's explicit yes in his own message.
- The roadmap (`docs/roadmap.md`) holds the plans. It says "ideas, not promises", and plans change often. Update it when plans change.

## Layout

- `src/StoryHeart.Core`: library file, folder scanning, chapters, book details from tags (TagLib#), update feed, naming. No Windows code; keep it that way so a web version could reuse it.
- `src/StoryHeart`: the WPF app. `MainWindow` is split into `.Library.cs`, `.Player.cs` and `.Updates.cs`. Dialogs in `Dialogs/`, Windows integration in `Platform/`, audio (NAudio, SoundTouch) in `Audio/`.
- `tests/StoryHeart.Core.Tests`: xUnit tests for Core.
- `installer/StoryHeart.iss`: Inno Setup, saved as UTF-8 with BOM. Never change `AppId`: it's how updates replace the old install (including BookHeart installs).
- `docs/manual.md` (the user manual, shipped as HTML), `docs/roadmap.md`, `release-notes/v<version>.md`, `legal/` (EULA, third-party notices, LGPL text).
- `site/`: the storyheart.app website, plain static HTML and CSS. Vercel project `storyheart` deploys it automatically when something in `site/` changes on `main`.
- `web/prototype`: the original web player the app started from.

## Building and testing

- `dotnet test tests/StoryHeart.Core.Tests` runs the tests. `dotnet build StoryHeart.sln -c Release` builds everything, and also works on Linux (`EnableWindowsTargeting`), though the app only runs on Windows.
- `pwsh build/build.ps1` (Windows) runs the tests, publishes the installed and portable builds, checks each one starts (`--smoke-test`), builds the manual and the installer into `release/`. GitHub Actions runs it on every push.
- The smoke test only checks the main window opens. Dialogs, the installer and updating need trying on Windows; say so when a change can't be checked here.

## Releasing

- Set the version in `Directory.Build.props`, write `release-notes/v<version>.md`, push, wait for the Build workflow to pass, then run the **Release** workflow with the version. A version with a dash (`0.2.0-beta.9`) is published as a pre-release, offered only to people with "Include test versions" ticked.
- Release notes: a short intro line, then `## What's new` with one change per list item. The update window shows only those list items, read out one per line. Testing notes go under a separate heading.
- Tags and branches can't be deleted directly from the cloud; use the **Delete release** and **Delete branch** workflows.
- The updater reads GitHub releases (`UpdateFeed.ReleasesUrl`) and needs `-Setup.exe` and `-Portable.zip` assets, checked by their SHA-256 digests. Old versions keep checking GitHub, so keep publishing there.

## Accessibility conventions

- Every control has a name a screen reader can read (`AutomationProperties.Name` or a `Label` with `Target`), and shortcuts go in `AutomationProperties.AcceleratorKey`.
- After every action, say what happened through `Announcer` (`_say.Say(...)`). It's also shown in the status line.
- Lists use `Row` (title, detail, and the spoken text) with the `RowItem` style. Typing letters jumps to an item, and Tab lands on the selected item.
- Arrow keys do nothing on buttons (`Dialogs.IgnoreArrowsOnButtons`); Tab moves between controls.
- Dialogs go through `Dialogs.Prepare`, which also adds tooltips to buttons (shown on mouse hover only, never on keyboard focus).
- Sections of a dialog are `GroupBox`es, so screen readers announce their names.
- Colours come from `Platform/Theme.cs` resources (light, dark and high contrast). Don't hard-code colours.

## Writing style

Plain, friendly, specific, British spelling (colour, licence, maximised). Short sentences, no marketing language, nothing that sounds machine-written. This applies to the app's text, the manual, release notes, the roadmap and the website.

## Legal

- The code is copyright James Trimble, all rights reserved (`LICENSE.txt`), published to read, not to reuse.
- `legal/EULA.txt` has a "Last updated" date. StoryHeart shows the agreement once when that date changes, so update the date whenever the agreement's wording changes.
- SoundTouch.Net and TagLib# are LGPL: they must stay separate DLLs, even in the single-file portable build (see `KeepLgplLibrarySeparate` in `StoryHeart.csproj`), and be listed in `legal/THIRD-PARTY-NOTICES.txt` and section 5 of the EULA. Any new third-party library goes in the notices too.
- StoryHeart collects nothing about its users. The only network use is checking for and downloading updates. Keep it that way unless the privacy wording is updated too.
