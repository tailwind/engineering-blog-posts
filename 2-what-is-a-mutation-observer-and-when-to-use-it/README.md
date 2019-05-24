# What is a Mutation Observer and when should you use it?

## What is is
Explain what it does - listens for any change in an element's contents
Explain when it came into being - link to the [mozilla doc](https://developer.mozilla.org/en-US/docs/Web/API/MutationObserver)

## When to use it
When you need to do things in response to an element's contents changing, and you don't care about *why* the content changed (don't care if it was a user event, a dom event, etc), it's a good opportunity to use a mutation observer.

## Real World Example – Case where Tailwind Uses It
Explain that we have a `contenteditable` div that changes:
1. When the user types
2. When the user selects a hashtag from the suggestions

...and that using `on.('change',...` doesn't work because the change handler isn't triggered by the user selecting a hashtag.

Show a stripped-down code snippet of the mutation observer being attached to the `contenteditable` div.

..........

blurb on how tailwind is hiring
