# Wordle-C

A very simple wordle clone in C.
It doesn't uses any cursor movement instructions or OS dependent functions.

It compiles in Borland Turbo C++ and runs under DOS.

## Quick introduction to Wordle

- You have to guess a 5 letter word
- You have at most 6 guesses
- The system tells you which characters in your guess are correct
- You cannot guess random letters, only words from a list

### Character hints

Wordle presents the character hints as follows:

- `_` This character does not exist in the word at all.
- `o` This character exists in the word, but is in the wrong place
- `#` This character is correct

The application is smart in regards to double letter guesses.
This means if you guess `ZORRO` when the word is `CRANE` it will show `__o__` and not `__oo_`

In addition to the basic rules,
this application also shows you which letters you haven't used yet.

Tip: If you already get correct letters after your first two guesses,
do not try to guess words with the letters in the correct places,
instead try to eliminate as many letters from the alphabet as possible within your first 3 guesses.

## Installation

1. Download the appropriate executable from the releases section, or [use gitload to simplify it](https://gitload.net/AyrA/Wordle-C)
2. Create a directory "list" and put the two word lists from the releases in there

## Usage

You're presented with a menu when you launch the application.

Note: This is a console application.
You can double click to run it, but it will close immediately when the game ends.
This means if the game ends on the 6th turn you don't know if you won or lost.

## Game Id

This game presents you with a game id. The id can be used to load the same word again later.
The id is simply the line number in `SOLUTION.TXT`.
Consider randomizing the word order in the solution list,
or people can simply look up the solution with the game id.

**CAUTION! Do not randomize ALL.TXT or the game will break**

## ALL.TXT vs SOLUTION.TXT

`ALL.TXT` contains a list of all accepted words (over 10k words).
The `SOLUTION.TXT` contains the list of all possible words that can be a solution (approx 2k words).

## Speed vs Memory

This version is written to run on pretty much anything that has a terminal.

- It requires almost no memory at all
- It has at most 3 file pointers open
- It doesn't uses OS dependent functions in the terminal, just basic text I/O

The application can save a lot of memory by not loading the entire lists into memory.
Instead, the lists are searched during gameplay as needed.

The downside of this approach is that searching for words is rather slow as it has to run through the entire file.
On an 4 MHz 8088 it can take up to 10 seconds to scan the list.
The list is alphabetical, so "CRANE" will be found faster than "ZORRO".

Various functions have been written to fix these issues.

Note: I do not know if the game would actually run on an 8088.
The speed figures were obtained by running DOSBox at that speed, but DOSBox emulates a better CPU.
The precompiled executable runs in an 8086 according to DOSBox though.

## Speed optimizations

Speed optimizations are important for games that are intended to run on old or undersized machines.

The two most important things are described below.

### Basic checks

This is fairly trivial.
First of all, we do not need to scan the word list if the guess is correct.
At this point we can immediately terminate the application with a success message.

The next optimization is to check the word length.
No need for further processing at all if the length is invalid.
Words with invalid length are not counted as guesses.

The final optimization is to check if the input is made up of the characters a-z only.
If not, no need to scan the list. This is not counted as a guess either.

### Cache

This is the most important optimization.
A naive implementation of the game would have to scan the complete word list for every guess.
It also has to scan the solution word list to count the number of possible words for the game id.

A simple caching mechanism is used that takes up 108 bytes.
It uses the fact that the complete word list is alphabetical.

When no cache exists, the complete wordlist is scanned for words.
The words themselves are not of interest, only the first character.
The cache can hold all characters A-Z plus an extra slot,
and for each character the file position of the first word is stored.

This means if a user guesses `HELLO`, we do not need to scan the entire list,
but instead can seek to the first word that begins with `H`,
and stop scanning the list if either `HELLO` is found, or the word no longer starts with `I`.
This cuts the scan time approximately by a factor of 26.

The final extra slot of the cache is used to store the word count in the solution list.

This means once the cache is built, the game starts up instantly.

The result of this is:

- The solution list only needs to be scanned when a game is started to pick the word but not to count words.
- The full word list is only ever scanned for words with the same letter as the current guess.

## Running on old machines

Various floppy disk images are provided with the game and word lists on them.

Sizes: 360k, 720k, 1.4M

## Compiling

The application should compile pretty much on anything.

Currently tested are:

- Windows 3.11
- Windows 10
- MS-DOS 6.22

Note that all these systems use CRLF as line break.
If you compile for linux,
you may need to adjust the line breaks in the word lists for the files to work properly.
The C file already has an EOL constant that adapts automatically in most cases.

If you plan on providing this over text based network connection (for example a BBS system),
be advised that telnet like most text based protocols (HTTP, FTP, SMTP, etc) also uses CRLF,
even on Linux.

- LF withour CR: Cursor moves down a line but not back to the start. Lines look like "stairs"
- CR without LF: Cursor moves to the start but not down. Lines overwrite each other
