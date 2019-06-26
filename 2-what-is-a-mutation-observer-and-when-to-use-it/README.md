# What is a Mutation Observer and when should you use it?

## When to use it
When you need to do things in response to an element's contents changing, and you don't care about *why* the content changed (don't care if it was a user event, a dom event, etc), it's a good opportunity to use a mutation observer.

## What is is
Explain what it does - listens for any change in an element's contents
Explain when it came into being - link to the [mozilla doc](https://developer.mozilla.org/en-US/docs/Web/API/MutationObserver)
## Using a MutationObserver in the Real World

At Tailwind, our members can schedule Instagram posts, including a caption:
![tailwind-post-example][luck]

As you can see, a Tailwind member can edit the post caption directly, as well as choose hashtags from the "Suggested Hashtags" section below the post.

Let's imagine that we want to add a character count to the post caption:

![character-count][fake-char-count]

We could try to update the character count as the user types by creating an `onchange`, like so:
```js
/**
 * When the user edits the caption, update the
 * character count box with the new caption length.
 */
function onCaptionChange (event) {
    const newCaptionLength = event.target.value.length;
    document.querySelector('.post-caption').innerHTML = newCaptionLength;
}
```

Unfortunately, this solution doesn't work for us.

The reason? The post caption changes when edited directly, but it also changes when a user selects a suggested hashtag!

If a hashtag is inserted into the post caption like so:

![insert-hashtag][insert-hashtag]

...then the `onCaptionChange` handler defined above won't be triggered, because the user isn't editing the caption directly.

What we need is something that will allow us to listen for *any* changes to the post caption, regardless of how those changes are triggered.

### Enter the MutationObserver

To see how a `MutationObserver` can let us watch for changes to a post's caption, we'll be using a jsfiddle which you can find [here](https://jsfiddle.net/yr4v5psc/8/):

![jsfiddle][jsfiddle]

Try playing with it. Create a caption, insert some hashtags using the buttons on the page, and watch the Character count change.

![jsfiddle-gif][jsfiddle-gif]

Here's how the this code sample works at a high-level:
1. Our post caption is a contenteditable div where the user creates a caption by typing and/or selecting hashtags to insert.
2. Any time the post caption changes, the character count below the caption is updated.

Now, let's see how we're using a `MutationObserver` to watch for changes.

#### Diving Into the Code

TODO

## blurb on how tailwind is hiring


[luck]: http://i67.tinypic.com/59n3p.png
[fake-char-count]: http://i68.tinypic.com/2dcakol.png
[insert-hashtag]: http://i68.tinypic.com/x6khs7.gif
[jsfiddle]: http://i65.tinypic.com/ji2txy.png
[jsfiddle-gif]: http://i63.tinypic.com/wjhw5h.gif
