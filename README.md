# Scripture
``scripture``: a simple SM-2 implementation

## Installation
```sh
curl -o /usr/local/bin/scripture https://raw.githubusercontent.com/michaelskyba/scripture/master/scripture
chmod +x /usr/local/bin/scripture
```
If /usr/local/bin is absent in $PATH, change the download location accordingly.

## Usage
Start a review session: ``scripture <deck file>``

A deck file is simply a TSV, with the columns (Front, Back, n, EF, I, last review). See ``sample_deck.tsv`` for an example. Note that **you should only provide the first two columns when creating cards**. The rest is data used by SM-2 , and will be generated automatically by scripture. Do not try to fill them with arbitrary numbers.

The delimiter can be changed via the $SCRIPTURE_DELIMITER environment variable. For example, use ``export SCRIPTURE_DELIMITER=","`` if you prefer CSV over TSV. The delimiter **must only be one character in length**.

Please open an issue if you need a bit more clarification on anything, I would be happy to provide assistance.

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
(Besides modifying the source code, which isn't onerous) ``scripture`` can be extended through $SCRIPTURE_HOOK, which functions in a similar fashion to Kiss's $KISS_HOOK. It will be called using the equivalent of ``$SCRIPTURE_HOOK type front_of_card back_of_card``. The "types" are as follows:
- "front-show" - called just after the front of a card has been shown to the user
- "back-show" - called just after the back of a card has been shown to the user
- "card-complete" - called just after the user has given themselves a grade

Here's an example, demonstrating how you could create cards that spawn images.
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
# (front isn't used for this hook)
back=$3

case $type in
	back-show) [ "${back%%:*}" = img ] && sxiv "${back#img:}" ;;
	card-complete) killall sxiv ;;
esac
```
```
/home/michael/deck.tsv
--------------------------------------------------
What does Bulgaria's flag look like?	img:/home/michael/bulgaria.png
What does Canada's flag look like?	img:/home/michael/canada.png
```
