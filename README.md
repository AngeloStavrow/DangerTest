# README / Danger "Getting Started" Friction Log

It's not even clear to me that this works for an iOS / macOS app, or if it's just for Swift Packages, so that's why I created a single-view iOS app called **DangerTest** to test this out.

## [Installation]

Followed the "Swift PM" version of the installation instructions. Had to create a file called **Dummyfile.swift** in my iOS project that _only_ imports Foundation, and then created this **Package.swift** file based on the sample code in the docs:

```swift
// swift-tools-version:4.2
import PackageDescription

let package = Package(
    name: "DangerTest",
    dependencies: [
      .package(url: "https://github.com/danger/swift.git", from: "1.0.0")
    ],
    targets: [
        // This is just an arbitrary Swift file in our app, that has
        // no dependencies outside of Foundation, the dependencies section
        // ensures that the library for Danger gets build also.
        .target(name: "dangertest", dependencies: ["Danger"], path: "DangerTest", sources: ["Dummyfile.swift"]),
    ]
)
```
Should I be using a newer version of `swift-tools-version`? Swift is at 5.2 but I don't know if swift-tools is versioned in lockstep. Not sure if "`dangertest`" is the right name for a target; at least in the iOS app project in Xcode, target names use uppercased words (e.g., DangerTest). Is this supposed to be the same target as my iOS app? Is this a different target that will be built later? I don't know.

`% swift build` seems to complete without issue. Moving on!

## [Creating a Dangerfile]

`% [swift run] danger edit` fails with:

```bash
zsh: bad pattern: [swift
```

I kinda figured the `[]` brackets would cause an error. I'm not sure why they're there. Is `swift run` optional? Is it supposed to be implied? I _think_ this is dependent on whether I installed via SwiftPM or Brew.

`% swift run danger edit` fails with:

```bash
error: no executable product named 'danger'
```

Interesting. Didn't I just install that? The installation section says I should be able to run `swift run danger-swift [cmd]`, maybe that's a typo in the docs.

`% swift run danger-swift edit` seems to work because it runs a bunch of stuff in the terminal, then launches a new Xcode project called Dangerfile. I'm then told, "to get yourself started, add this to the Dangerfile.swift in the Xcode project"… but there _is_ no **Dangerfile.swift** file in the project. There's a **Package.swift**, and there's **Sources/Dangerfile/main.swift**, but that's about it. Should I create new file? Looking through the linked document "[About The Dangerfile]", it _looks_ like the format for my Dangerfile matches what's in **Sources/Dangerfile/main.swift**, so let's see what happens if I add the sample code here.

I save the changes to **Sources/Dangerfile/main.swift** and then see in the terminal that I just need to press the return key once I'm done, so I do that and I'm returned to the prompt. It looks like that worked, because I now have a **Dangerfile.swift** in the root directory that contains the sample code, so that's good!

## [Testing Locally]

I don't have an existing PR available since this is a brand new iOS project, but the instructions here are really helpful for this exact situation. Here goes:

```bash
% swift run danger-swift pr https://github.com/danger/swift/pull/146
ERROR: commandFailed(command: "which danger", exitCode: 1, stdout: "danger not found\n", stderr: "")
``` 

Huh. Now what? Let's dig into the GitHub issues, where I find [this open issue] that suggests I need to have Danger JS installed before I can run commands, maybe — this leads me to [this PR] that updates the README to include installation steps for Danger JS. So let me try that and then run the above local test again.

`% npm install -g danger` completes without issue, so I try running the local test again, and it seems to work:

```bash
% swift run danger-swift pr https://github.com/danger/swift/pull/146
Starting Danger PR on danger/swift#146
You don't have a DANGER_GITHUB_API_TOKEN set up, this is optional, but TBH, you want to do this
Check out: http://danger.systems/js/guides/the_dangerfile.html#working-on-your-dangerfile


Danger: ✓ passed, found only messages.
## Messages
These files have changed: CHANGELOG.md, Sources/DangerFixtures/DangerFixtures.swift, Tests/DangerTests/XCTestManifests.swift, Sources/Danger/Plugins/SwiftLint/CurrentPathProvider.swift, Sources/Danger/Plugins/SwiftLint/ShellExecutor.swift, Sources/Danger/Plugins/SwiftLint/SwiftLint.swift, Sources/Danger/Plugins/SwiftLint/SwiftLintViolation.swift, Tests/DangerTests/SwiftLint/DangerSwiftLintTests.swift, Tests/DangerTests/SwiftLint/FakeCurrentPathProvider.swift, Tests/DangerTests/SwiftLint/FakeShellExecutor.swift, Tests/DangerTests/SwiftLint/URL+Utils.swift, Tests/DangerTests/SwiftLint/ViolationTests.swift
```

Cool!

## [Creating a bot account for Danger to use]

This is optional but recommended. Do I need to do this if I intend on using Danger Swift as a GitHub Action? I don't want to pay to add a bot account to my team if I don't have to, so I'm going to try skipping this for now and see what happens.

## [Setting up Danger to run on your CI]

Following the instructions to use GitHub Actions here after skipping the creation of a bot account. I'll set it up as its own action, rather than making Danger as another build step. I don't really know what the pros and cons are of either, but right now it seems like the simplest solution. I replace the boilerplate YAML with what's in the docs:

```yaml
name: "Danger Swift"
on: [pull_request]

jobs:
build:
name: Danger JS
runs-on: ubuntu-latest
steps:
- uses: actions/checkout@v1
- name: Danger
uses: danger/swift@2.0.3
env: GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

Searching for "Danger Swift" in the GitHub Actions marketplace suggests I use this instead:

```yaml
- name: Danger Swift
  uses: danger/swift@2.0.3
```

So I replace those lines in the file and hit preview… and see a preview of the same file on GitHub, so that's not super helpful. Is it validating anything? Would I see an error if there was one?

According to the Danger Swift docs, "you can use the [auto-generated] `GITHUB_TOKEN` to authenticate in a workflow run. Using this token will post the danger comment as the github-actions app user." So that suggests I was right about being able to skip the creation of a bot account. I commit the GitHub Actions YAML file to the `master` branch and I'm ready for the next step.

## [Verify Installation]

I should be able to push some commits and have GitHub Actions do something — I guess I'll need to open a PR to trigger Danger. Let's see what happens!

I checkout a new branch with all the Danger stuff added in the local repo and add my changes — immediately getting a warning:

```bash
% git add .
warning: adding embedded git repository: .build/checkouts/Files
hint: You've added another git repository inside your current repository.
hint: Clones of the outer repository will not contain the contents of
hint: the embedded repository and will not know how to obtain it.
hint: If you meant to add a submodule, use:
hint: 
hint: 	git submodule add <url> .build/checkouts/Files
hint: 
hint: If you added this path by mistake, you can remove it from the
hint: index with:
hint: 
hint: 	git rm --cached .build/checkouts/Files
hint: 
hint: See "git help submodule" for more information.
warning: adding embedded git repository: .build/checkouts/Logger
warning: adding embedded git repository: .build/checkouts/Marathon
warning: adding embedded git repository: .build/checkouts/Releases
warning: adding embedded git repository: .build/checkouts/RequestKit
warning: adding embedded git repository: .build/checkouts/Require
warning: adding embedded git repository: .build/checkouts/ShellOut
warning: adding embedded git repository: .build/checkouts/octokit.swift
warning: adding embedded git repository: .build/checkouts/swift
```

Trying to follow the instructions fails:

```bash
% git rm --cached .build/checkouts/Files
error: the following file has staged content different from both the
file and the HEAD:
    .build/checkouts/Files
(use -f to force removal)
```

I guess I should have added .build to a .gitignore file or something. Oh well, this is just a test repo, so that's fine. Let's continue. I commit my changes and push them up to GitHub on a feature branch.

Once that's done, I open a PR on GitHub… and wait. Nothing seems to happen and so I go to the GitHub Actions tab and see that it did run a workflow when I committed the YAML file, but it failed:

> Check failure on line 12 in .github/workflows/main.yml
> 
> **GitHub Actions**
> **/ .github/workflows/main.yml**
> 
> Invalid workflow file
> You have an error in your yaml syntax on line 12

Nice. That's the `env:` line which, looking back, should have no value, and the `GITHUB_TOKEN` stuff should be on its own line. That's my fault, I think I missed the return key when adding that! I change it and go back to the GitHub Actions tab — and it still fails:

> Check failure
> 
> **GitHub Actions**
> **/.github/workflows/main.yml**
> 
> Error
> No jobs defined in `jobs`

Blerf?

Maybe it's a whitespace/indentation error. Looking in the GitHub issues for Danger Swift, I see [this file], so I fix up my YAML file to use the same — this matches what I see in the [README] (which I haven't looked at, since I'm going through the guide on the site):

```yaml
name: "Danger Swift"

on: [pull_request]

jobs:
  build:
    runs-on: ubuntu-latest
    name: "Run Danger"
    steps:
      - uses: actions/checkout@v1
      - name: Danger
        uses: danger/swift@2.0.3
        with:
            args: --failOnErrors --no-publish-check
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

Let's see if that sorts things out. I close my old PR and open a new one aaaaand… it looks like the action is running! Building Danger takes a while (4m30s) and then… fails?

On the "Danger" step of the job, I have:

```
Error:  { Error: write EPIPE
    at afterWriteDispatched (internal/stream_base_commons.js:78:25)
    at writeGeneric (internal/stream_base_commons.js:73:3)
    at Socket._writeGeneric (net.js:714:5)
    at Socket._write (net.js:726:8)
    at doWrite (_stream_writable.js:415:12)
    at writeOrBuffer (_stream_writable.js:399:5)
    at Socket.Writable.write (_stream_writable.js:299:11)
    at sendDSLToSubprocess (/github/home/.npm/_npx/1/lib/node_modules/danger/distribution/commands/utils/runDangerSubprocess.js:89:25)
    at Object.exports.runDangerSubprocess (/github/home/.npm/_npx/1/lib/node_modules/danger/distribution/commands/utils/runDangerSubprocess.js:100:5)
    at Object.<anonymous> (/github/home/.npm/_npx/1/lib/node_modules/danger/distribution/commands/ci/runner.js:112:39) errno: 'EPIPE', code: 'EPIPE', syscall: 'write' }
```

I _think_ this might be because there's no Dangerfile in the repo yet? I'll merge this in, make a trivial commit, and try again.

But, nope, that doesn't help. So now I'm stuck. Technically, I've added Danger to the repository, but it… doesn't work for some reason that I don't fully understand.

[Installation]: https://danger.systems/swift/guides/getting_started.html#swift-pm
[Creating a Dangerfile]: https://danger.systems/swift/guides/getting_started.html#creating-a-dangerfile
[About The Dangerfile]: https://danger.systems/swift/guides/about_the_dangerfile.html
[Testing Locally]: https://danger.systems/swift/guides/getting_started.html#testing-locally
[this open issue]: https://github.com/danger/swift/issues/336
[this PR]: https://github.com/danger/swift/pull/337/files
[Creating a bot account for Danger to use]: https://danger.systems/swift/guides/getting_started.html#creating-a-bot-account-for-danger-to-use
[Setting up Danger to run on your CI]: https://danger.systems/swift/guides/getting_started.html#setting-up-danger-to-run-on-your-ci
[Verify Installation]: https://danger.systems/swift/guides/getting_started.html#verify-installation
[this file]: https://github.com/danger/swift/issues/316#issuecomment-582596405
[README]: https://github.com/danger/swift/blob/master/README.md
