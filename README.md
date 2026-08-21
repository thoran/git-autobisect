# git-autobisect

## Description

Automate git bisect.

`git bisect` needs a bad revision and a good one, and the good one is usually the tedious part: you know the test fails now, but not when it last passed. `git-autobisect` finds it for you, from the history of the test file itself:

1. Takes the revision you are standing on as the bad one.
2. Walks the revisions of the test file, newest first, checking each out and running the test command until one passes. That revision becomes the good one.
3. Runs `git bisect run` with your test command between the two.
4. Resets the bisect and checks the revision you started on back out.

## Installation

### 0. Have a recent version of Ruby installed

### 1a. Via Homebrew

```shell
$ brew tap thoran/tap
$ brew install thoran/tap/git-autobisect
```

### 1b. Manually

```shell
$ git clone https://github.com/thoran/git-autobisect
$ cp ./git-autobisect/bin/git-autobisect to your preferred executable path
$ chmod +x /path/to/git-autobisect
```

## Dependencies

[switches.rb](https://github.com/thoran/switches.rb), for the command line switch, vendored by the formula along with ostruct, which switches.rb brings with it.

## Usage

### 0. Stand on the revision where the test fails

That revision is taken as the bad point, so check it out before starting. Nothing asks you to confirm it.

### 1. Give it the test command

```shell
$ git-autobisect bundle exec rspec spec/models/class.rb
```

Everything left on the command line after the switches are taken out is the test command, so it needs no quoting.

### 2. Where something has to run before each test run

```shell
$ git-autobisect bundle exec rspec spec/models/class.rb -p 'bundle install'
$ git-autobisect bundle exec rspec spec/models/class.rb --preparatory_command 'bundle install'
```

Gems which move between revisions are the usual reason: without `bundle install` first, the test run fails for want of a gem rather than for the reason you are bisecting.

### Options

| Option | Takes | What it does |
| --- | --- | --- |
| `-p`, `--preparatory_command` | a command | Runs before each test run while the good revision is being looked for |

## What it does to your repository

In order:

1. `git bisect reset`, so an unfinished bisect is cleared away.
2. `git bisect start`.
3. `git bisect bad <the revision you were on>`.
4. Then, to find the good one, it walks `git log --format=%H <test file>` — the revisions of the test file itself, newest first — checking each out, running the preparatory command where given, and running the test command. The first that passes becomes `git bisect good <that revision>`.
5. `git bisect run <test command>`.
6. `git bisect reset`, then `git checkout <the revision you were on>`, so you finish where you began.

## Notes

1. The test file is taken as the last word of the command, with anything after a colon dropped, so `spec/models/class.rb:42` names the same file. Keep the file last in the command.
2. Only revisions in which the test file itself changed are tried as candidates for the good point. Where the test has sat untouched across the breakage, the search has few places to look.
3. The preparatory command runs during that search, but **not** during `git bisect run`, which is issued with the test command alone. Where the preparatory command is what makes the test runnable, the bisect proper may fail for the reason the switch exists to prevent.
4. Revisions are checked out one after another, so start from a clean working tree.
5. Any bisect already under way is reset before this one starts, without asking.
6. Where no revision of the test file passes, it says so and stops rather than guessing.

## Contributing

1. Fork it: `https://github.com/thoran/git-autobisect/fork`
2. Create your feature branch: `git checkout -b my-new-feature`
3. Commit your changes: `git commit -am 'Add some feature'`
4. Push to the branch: `git push origin my-new-feature`
5. Create a new pull request


## License

MIT
