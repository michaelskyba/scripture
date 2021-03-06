# Scripture
``scripture``: a simple SM-2 implementation

## Archive
I made [tunnel](https://github.com/michaelskyba/tunnel), a new SM-2 implementation
which is better than scripture in pretty much every way. Thus, I don't see any
reason for anybody to use scripture anymore. If you are one of the zero people
who is using scripture and agrees to switch to tunnel, be aware:

- "tunnel" isn't interactive like scripture is and therefore requires a wrapper
instead of using a hook system. If you liked the way scripture worked, you can
use shovel, a provided example wrapper, which works similarly.
- tunnel forces TSV for deck files and has no way of configuring the delimiter.
If you are using/prefer e.g. CSV, include conversions like ``sed "s/,/\t/g"``
in your wrapper script.
- tunnel uses Unix time for the last review date instead of YYYY-MM-DD.
- tunnel doesn't allow decimals for inter-repetition intervals (fifth column).

To use your existing decks with tunnel, you'll need to use a script that
iterates over each line, converting each date to Unix time and each
inter-repetition number to an integer.
[Here](https://gist.github.com/michaelskyba/84d054b583d5a98780f4fea7ac67c216)
is an example that prints to stdout.

## Usage
Start a review session: ``scripture <deck file>``

A deck file is simply a TSV with the columns (Front, Back, n, EF, I, last
review). See ``sample_deck.tsv`` for an example. Note that **you should only
provide the first two columns when creating cards**. The rest is data used by
SM-2 and will be generated automatically by scripture. Do not try to fill them
with arbitrary numbers.

Lines with exactly zero instances of the delimiter will be ignored, which
allows line breaks and comments.

The delimiter can be changed via the $SCRIPTURE_DELIMITER environment variable.
For example, use ``export SCRIPTURE_DELIMITER=","`` if you prefer CSV over TSV.
The delimiter **must only be one character in length**.

### Reviewing
In each review session, ``scripture`` will show you the "front" of your card
and you are to attempt to recall the "back". After doing so, you will press
Enter to reveal the answer and be prompted to grade yourself (0-5), with each
grade having the following meaning:

- 0: "Total blackout", complete failure to recall the information.
- 1: Incorrect response, but upon seeing the correct answer it felt familiar.
- 2: Incorrect response, but upon seeing the correct answer it seemed easy to remember.
- 3: Correct response, but required significant difficulty to recall.
- 4: Correct response, after some hesitation.
- 5: Correct response with perfect recall.

See https://en.wikipedia.org/wiki/SuperMemo for more information.

### Extending
(Besides modifying the source code, which is not onerous) ``scripture`` can be
extended through $SCRIPTURE_HOOK, which functions in a similar fashion to KISS
Linux's $KISS_HOOK. Alternatively, put a "hook" file in the directory you're
calling ``scripture`` from. Your script will be passed several arguments, the
first being the type of hook, and the rest depending on which type you get.

| hook type | arguments |
| ---- | --------- |
| review_start | hook_type deck_file |
| front_show | hook_type front_of_card back_of_card |
| back_show | hook_type front_of_card back_of_card |
| card_complete | hook_type front_of_card back_of_card grade |
| pre_iteration | hook_type review_file |
| review_complete   | hook_type deck_file |

Hooks are not called with "&" at the end. See the example below for how to work
around it, in the case that it may seem necessary.

Here is an example, demonstrating how you could create cards that spawn images.
Note that your hook script can be written in any language, the only requirement
being that it is executable.

```sh
/etc/zshenv
--------------------------------------------------
export SCRIPTURE_HOOK=/home/michael/s_images
```
```sh
/home/michael/s_images
--------------------------------------------------
#!/bin/sh

type=$1
# (front is not used for this hook)
back=$3
# $3 will not always exist, but $back will not be called unless it does

case $type in
	back_show) [ "${back%%:*}" = img ] && sxiv "${back#img:}" & ;;
	card_complete) killall sxiv ;;
esac
```
```
/home/michael/deck.tsv
--------------------------------------------------
What does Bulgaria's flag look like?	img:/home/michael/bulgaria.png
What does Canada's flag look like?	img:/home/michael/canada.png
```

#### scripture_hide
Both the front_show and back_show hooks are called before their actions. In
these hooks, you can create ``/tmp/scripture_hide`` to tell scripture not to
show the respective side of the card. See [my current personal
hook](https://gist.github.com/michaelskyba/8d4d68387a5ecd6bdce1ed5bf7a61939)
for an example of how this can be useful.

### Restarting a review
Before your review starts, scripture makes a copy of your deck in
/tmp/scripture, deleting it once you successfully complete your review. The
idea is that if something goes wrong, like you accidentally cancel a review
while have cards you graded < 4, or mark a card wrong by accident, you can
restore it effortlessly.

The name of the file will be YYYY-MM-DD-Epoch, e.g.
/tmp/scripture/2021-09-23-1632412533. Once you've found the file, overwrite
your current deck file with it, and run scripture to start the review again.

## macOS and BSD
``scripture`` requires GNU/BusyBox sed and GNU/BusyBox date. I will provide
instructions for getting those on macOS. If you're on BSD and want to use
scripture, adapt these steps accordingly.

1. Install [Homebrew](https://brew.sh/)
2. ``brew install coreutils``
3. ``brew install gnu-sed``
4. In the ``scripture`` script, replace all uses of the ``date`` _command_ with
"gdate", and replace all uses of the ``sed`` command with "gsed".

## Similar Projects
- https://apps.ankiweb.net/
- https://github.com/proycon/vocage
- https://github.com/0x766F6964/oboeru
- https://mnemosyne-proj.org/
- https://gitlab.com/phillord/org-drill/

## Bugs
Open an issue on GitHub if you find a bug. Also open an issue if you need a bit
of clarification on anything. I would be happy to provide assistance. You can
also email me, but I would prefer GitHub because that way, other people can see
your issue and possibly benefit.
