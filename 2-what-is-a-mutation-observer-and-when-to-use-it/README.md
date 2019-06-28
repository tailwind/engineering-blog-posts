# What is a Mutation Observer and when should you use it?

The `MutationObserver` is a powerful but lesser-known concept that is supported by all major browsers. It allows you to watch for changes in the DOM.

With it, you can observe:
1. When an element is inserted or removed
2. When an element is modified (both its attributes and content)
3. When an element's child/children are modified

Cool! But when is it useful?

## Using a `MutationObserver` in the Real World

At Tailwind, we recently had a case where a `MutationObserver` was the perfect tool for the job.

For context, our members use Tailwind to schedule their Instagram posts. They often include captions with their posts, like so:

![tailwind-post-example][luck]

Our members can edit their post captions directly, as well as select hashtags from the "Suggested Hashtags" section below the post.

Let's imagine that we want to add a character count to the post caption:

![character-count][fake-char-count]

If the post caption was a simple textbox where users typed the caption, we could use an `onchange` handler to listen for changes and update the character count, like so:
```js
/**
 * When the user edits the caption, update the
 * character count element with the new caption length.
 */
function onCaptionChange (event) {
    const newCaptionLength = event.target.value.length;
    document.querySelector('.post-caption').innerHTML = newCaptionLength;
}
```

Unfortunately, this solution doesn't work for us.

The reason? The post caption can edited directly, but it also changes when a user selects a suggested hashtag!

If a hashtag is inserted into the post caption like so:

![insert-hashtag][insert-hashtag]

...then the `onCaptionChange` handler defined above won't be triggered, because the user isn't editing the caption directly. In general, `onchange` handlers are not triggered when an element is programmatically changed, but rather when the user interacts with the element. When a Tailwind member selects a hashtag, we programmatically insert it into the post caption, so the `onCaptionChange` has no opportunity to fire.

What we need is something that will allow us to listen for *any* changes to the post caption, regardless of whether those changes come from the user typing in the caption or selecting a hashtag.

### Enter the `MutationObserver`

To see how a `MutationObserver` can let us watch for changes to a post's caption, we'll be using a jsfiddle example which you can find [here](https://jsfiddle.net/Ly03mbs1/5/).

![jsfiddle][jsfiddle]

Try playing with the link above. Create a caption, insert some hashtags using the buttons on the page, and watch the Character count change like so:

![jsfiddle-gif][jsfiddle-gif]

### Diving Into the Code

Here's how the [the code](https://jsfiddle.net/Ly03mbs1/5/) works at a high-level:
1. Our post caption is a `contenteditable` div where the user creates a caption by typing and/or selecting hashtags to insert.
2. Any time the post caption changes, the character count below the caption is updated.

Now, let's break the code down piece by piece to see how we're using a `MutationObserver` to update the character count.

**The editable caption**

We use a `contenteditable` div to allow users to edit the caption. Here's how that looks in the DOM:
```html
<div class="post-caption" contenteditable="true" placeholder="Enter your caption here..."></div>
```
![editable-caption][editable-caption]

**The caption character count**

We'll use a simple span to show the character count for the caption:
```html
<span>Character count: </span><span id="character-count">0</span>
```
![char-count-span][char-count-span]

(it starts at `0` because the caption is empty before the user starting adding content)


**The suggested hashtags**

A user can add suggested hashtags by clicking the buttons provided. In production, Tailwind crunches a bunch of data to tell users about the best hashtags to add to a post's caption, but for this example we're just using simple static buttons that look like this:
```html
<input type="button" value="#football" onclick="addTextToDiv('#football')" />
<input type="button" value="#luck" onclick="addTextToDiv('#luck')" />
<input type="button" value="#nfl" onclick="addTextToDiv('#nfl')" />
```
![mock-suggested-hashtags][mock-suggested-hashtags]

Notice the `addTextToDiv(...)` handlers? That function is responsible for adding a hashtag to the post's caption when the user clicks on a hashtag button. It looks like this:

```js
// Add text to the contenteditable div
const postCaptionElement = document.querySelector('.post-caption');

...

function addTextToDiv(hashtag) {
  postCaptionElement.innerText += ` ${hashtag}`;
}
```

It's a straightforward function that takes the post caption element and append the hashtag to the end of the caption (after whatever text the user has already entered).

**Now, the fun part!**

Let's look at the `MutationObserver` and see how it listens for changes.

The `MutationObserver` is defined like so:
```js
// Create a mutation observer instance
const observer = new MutationObserver(function(mutations) {
  // For each mutation, update the character count <span>:
  mutations.forEach(updateCharacterCount);
});
```
This instnace of the `MutationObserver` is defined to listen for changes, and for each change, call the `updateCharacterCount` function, which looks this:
```js
const charCountElement = document.querySelector('#character-count');
function updateCharacterCount() {
  charCountElement.innerHTML = postCaptionElement.innerText.length;
}
```
Remember the `<span id="character-count">` defined above? The function above directly changes that element by looking at the post caption, getting the length of its content (IE how many characters are in the caption), and setting the `#character-count` element with the new number of characters.

Up till now, we've defined the `MutationObserver` and what it needs to do when the post caption changes. The last remaining step is to attach the `observer` variable created above to an actual element in the DOM:

```js
// Define the configuration for the observer
const config = {
  characterData: true,
  childList: true,
  subtree: true
};

const postCaptionElement = document.querySelector('.post-caption');

// Tell the observer what node to observe and what options to use
observer.observe(postCaptionElement, config);
```

The `observer` that we created earlier has a method called `observe`, that accepts two arguments:
1. The element or element to observe (in this case, the `postCaptionElement`)
2. A `config` object, which tells the `MutationObserver` what kinds of changes to watch for. There are bunch of options, which you can learn more about [here](https://javascript.info/mutation-observer). Breaking down the ones we use:

```js
const config = {
  // Watch for changes to the node's text content
  characterData: true,
  // Watch for changes to the node's direct child/children
  childList: true,
  // Watch for changes to all of the node's descendants
  subtree: true
};
```

As mentioned, there are other changes you might want to watch for, depending on our use-case. For the purposes of this example, only the content of the post caption (its text content and any children/descendants) are what we care about.

##### Tailwind is Hiring!

The best part of growing as a developer is learning about new/unknown concepts and how to apply them; something that we do at Tailwind every day. If this sounds interesting to you and you're looking for a job, [we'd love to talk](https://www.tailwindapp.com/careers)!

[luck]: https://user-images.githubusercontent.com/708562/60361012-17d80080-99ab-11e9-921c-7faec867fc81.png
[fake-char-count]: https://user-images.githubusercontent.com/708562/60361011-17d80080-99ab-11e9-9236-9c729b274faa.png
[insert-hashtag]: https://user-images.githubusercontent.com/708562/60361009-17d80080-99ab-11e9-86d2-370cff93ceb9.gif
[jsfiddle]: https://user-images.githubusercontent.com/708562/60361008-173f6a00-99ab-11e9-848c-5918bf44c6b4.png
[jsfiddle-gif]: https://user-images.githubusercontent.com/708562/60361007-173f6a00-99ab-11e9-9203-238845c08fce.gif
[editable-caption]: https://user-images.githubusercontent.com/708562/60359432-055bc800-99a7-11e9-9b3b-b87c4c3ae131.png
[char-count-span]: https://user-images.githubusercontent.com/708562/60359582-6be0e600-99a7-11e9-8e55-1df15699acc0.png
[mock-suggested-hashtags]: https://user-images.githubusercontent.com/708562/60359683-be220700-99a7-11e9-9f31-36e2296e901a.png
