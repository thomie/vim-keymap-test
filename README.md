## Goal
Check how closely the key bindings of
[evil-helix](https://github.com/usagi-flow/evil-helix) and other vim-alike
editors match those of Vim itself.

A simple test looks like this:

```
# 1. x                                 <--- comment
<
foo                                    <--- input
$ ./bin/keymaptest vim 'send x'        <--- commands to execute: `x`
oo                                     <--- expected output
>= 0                                   <--- expected exit code
```

This test first creates a tempfile containing the input `foo`. Then it
opens that file with `vim` and simulates a user pressing `x`. The test passes
if the contents of the tempfile matches the expected output.

A somewhat more complicated test looks likes this:
```
# 1. when exiting insert mode after appending, we should return to the
# character before the cursor
<
ab
$ ./bin/keymaptest vim 'send a' 'expect INS' 'send $ESC' 'expect NOR' 'send r1'
1b
>= 0
```
The `expect` commands are necessary only for tests that switch modes
(insert/select).

Currently vim and evil-helix are supported. The test runner is in
`bin/keymaptest`, the (minimal) vim and helix configuration files are in the
`config/` directory.

The test files are named after their Vim :help section (e.g. ``:help deleting`).


## Requirements
* [shelltestrunner](https://github.com/simonmichael/shelltestrunner)
* [expect](https://core.tcl-lang.org/expect/index)

Both might be available by your package manager:

* Ubuntu: ```sudo apt install expect shelltestrunner```

* Nix: ```pkgs.expect pkgs.haskellPackages.shelltestrunner```

## Usage

* To run all tests with `evil-hx`, use `-Dvim=<path-to-evil-hx>`:
```sh
$ cd vim-keymap-tests
$ sheltest . -Dvim=evil-hx
:./delete-insert.test:1: [OK]
:./delete-insert.test:2: [OK]
:./delete-insert.test:3: [OK]
...etc...

         Test Cases  Total 
 Passed  22          22    
 Failed  18          18    
 Total   40          40
```

(Tested with: evil-helix release-20250104 (c267dad0, helix 25.1))


* To only run test number `<n>` from a specific file, execute that file
  directly and supply `-i<n>`:
```sh
$ ./word-motions.test -Dvim=hx -i1
:./word-motions.test:1: [Failed]
Command (at line 7):
./bin/keymaptest hx 'send wr1' 'send wr,' 'send wr2' 'send wr3'
Expected stdout: 
foo 1ar,2a3

Got stdout:      
foo 1ar,baz2

         Test Cases  Total 
 Passed  0           0     
 Failed  1           1     
 Total   1           1     
```

* To run all tests with `vim`:

```sh
$ shelltest .
:./delete-insert.test:1: [OK]
:./delete-insert.test:2: [OK]
:./delete-insert.test:3: [OK]
...etc...

         Test Cases  Total 
 Passed  40          40    
 Failed  0           0     
 Total   40          40    
```
(Tested with: VIM - Vi IMproved 9.1)

## Limitations
* For some reason the helix tests are very slow. Helix hangs for 2 seconds on
every test. I don't know how to fix this, but it's workable for now.

* Tests that change mode (normal->insert->normal) are finicky to write. If you
get it wrong, the test will hang for 10 seconds or indefinitely. Tip: first get
it working with vim, then hx.

* Tests currently need to be written manually. It would be cool to generate
them automatically with something like
[QuickCheck](https://hackage.haskell.org/package/QuickCheck).

* Instead of spawning a new instance of Vim or Helix for every test, perhaps
it's possible to start a single instance and have each test run in a new
buffer (should be faster).
