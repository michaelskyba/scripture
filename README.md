# Scripture
``scripture``: a simple SM-2 implementation

## Dependencies
- BusyBox date
- bc

## Usage
Start a review session: ``scripture <deck file>``

A deck file is simply a TSV, with the columns (Front, Back, n, EF, I, last review). See ``sample_deck.tsv`` for an example. Note that **you should only provide the first two columns when creating cards**. The rest is data used by SM-2 , and will be generated automatically by scripture. Do not try to fill them with arbitrary numbers.

Lines with exactly zero instances of the delimiter will be ignored, which allows line breaks and comments.

The delimiter can be changed via the $SCRIPTURE_DELIMITER environment variable. For example, use ``export SCRIPTURE_DELIMITER=","`` if you prefer CSV over TSV. The delimiter **must only be one character in length**.

### Reviewing
In each review session, ``scripture`` will show you the "front" of your card and you are to attempt to recall the "back". After doing so, you will press enter to reveal the answer and be prompted to grade yourself (0-5), with each grade having the following meaning:
- 0: "Total blackout", complete failure to recall the information.
- 1: Incorrect response, but upon seeing the correct answer it felt familiar.
- 2: Incorrect response, but upon seeing the correct answer it seemed easy to remember.
- 3: Correct response, but required significant difficulty to recall.
- 4: Correct response, after some hesitation.
- 5: Correct response with perfect recall.

See https://en.wikipedia.org/wiki/SuperMemo for more information.

### Extending
(Besides modifying the source code, which is not onerous) ``scripture`` can be extended through $SCRIPTURE_HOOK, which functions in a similar fashion to KISS Linux's $KISS_HOOK. Alternatively, put a "hook" file in the directory you're calling ``scripture`` from. Your script will be passed several arguments, the first being the type of hook, and the rest depending on which type you get.

| hook type | arguments |
| ---- | --------- |
| review_start | hook_type deck_file |
| front_show | hook_type front_of_card back_of_card |
| back_show | hook_type front_of_card back_of_card |
| card_complete | hook_type front_of_card back_of_card grade |
| pre_iteration | hook_type review_file |
| review_complete   | hook_type deck_file |

Hooks are not called with "&" at the end. See the example below for how to work around it, in the case that it may seem necessary.

Here is an example, demonstrating how you could create cards that spawn images. Note that your hook script can be written in any language, the only requirement being that it is executable.
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

## Similar Projects
- https://apps.ankiweb.net/
- https://github.com/proycon/vocage
- https://mnemosyne-proj.org/
- https://gitlab.com/phillord/org-drill/

## Bugs
Open an issue on GitHub (or email me) if you find a bug. Also open an issue if you need a bit of clarification on anything. I would be happy to provide assistance.
