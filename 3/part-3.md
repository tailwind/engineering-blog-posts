# Improving an Existing React Codebase with Better State Management – Part 3

*This is part three of a three part series. Be sure to check out parts [one]() and [two]() [TODO link these]!*

In the [first part]() of this series, we covered the idea of propsing a new way for Tailwind to manage React state. This would involve changing Tailwind's codebase to use modern React concepts – actions, `connect()`, etc.

In [part two](), we covered the process of proposing this idea to the broader Tailwind engineering team, including gathering feedback and getting buy-in from the team.

Once the team bought into the idea of using the new pattern, the only item left was to actually begin to implement the new pattern in the codebase. My process for doing so took place in 3 steps:

1. **Opening a Small, Focused Pull Request**
1. **Getting PR feedback, then merging and deploying the change**
1. **Documenting the Change for Others**

Let's look at these steps in depth.

----

### 1. Opening a Small, Focused Pull Request

I wanted the first example of this new Redux pattern to be a small, simple, working proof-of-concept in the codebase.

With those constraints in mind, I landed on a simple component that we expose to Tailwind members, that allows them to insert Emojis into the caption of their posts.

![Tailwind's Emoji Picker](https://user-images.githubusercontent.com/708562/65169525-b2793600-da14-11e9-81cf-35bff452dcd0.png)

The emoji picker is built with React, but the post caption textbox is built in the jQuery portion of Tailwind's app. Meaning that when the user selects an emoji to insert into a post caption, the React app has to trigger a call to the jQuery app, using the global `window` object.

As far as the React app is concerned, this is an external call. Sounds like a good candidate for an action creator.

The code needs to call to a method that exists in the jQuery app, call `insertEmojiIntoPostCaption`, and tells it to insert the emoji into the post caption at a specific character index, like so:

![Inserting an Emoji](https://user-images.githubusercontent.com/708562/65169857-75617380-da15-11e9-99b9-bf2b5f791f3a.gif)

Inside of the Emoji Picker component, the code looks like this:

```js
import React, {Component} from 'react';

// Action Creator to an add emoji to a post caption that exists in Tailwind the jQuery app:
const addEmojiToDescription = (postId, emojiUnicode, caretLocation) => {
  return dispatch => {
    // Add emoji to caption
    window.PostDescriptions.insertEmojiIntoPostCaption(postId, emojiUnicode, caretLocation);
    // Inform react state that the emoji was added
    dispatch({ type: ACTIONS.ADD_EMOJI });
  };
};

// Emoji Picker Reusable Component
class EmojiPicker extends Component {
  ...
  
  addEmojiToPostCaption = (emoji) => {
    this.props.addEmojiToDescription(
      this.props.postId,
      emoji,
      this.props.cursorPosition
    );
  };
  
  render () {
    return (
      <div className='emoji-picker'>
        ...
        <button
          onClick={this.addEmojiToPostCaption}
        >
        ...
      </div>
    );
  }
}

// Connect the component to `addEmojiToDescription` action creator,
// so that it's available as a prop:
const mapDispatchToProps = {
  addEmojiToDescription
};

// Note: Since this emoji picker doesn't need much from Redux state,
// We're using `null` where we would normally use `mapStateToProps`:
const ConnectedEmojiPicker = connect(null, mapDispatchToProps)(EmojiPicker);
```

That's it! This PR was intentionally as small as possible so that it was easier to solicit reviews.

### 2. Getting PR feedback, then merging and deploying the change

Because the PR was so small and focused, I was able to use the exact same PR strategy that we always use at Tailwind:

1. Open a Pull Request
2. Perform a self review, fixing any issues you spot and leaving comments that will make ambiguties clear to future reviewers
3. Request review from two people on the team

Using that strategy, I two people on the team who are intimately familiar with React weighed in and provided some minor feedback.

After that, I merged and deployed the new Emoji Picker into production.

### 3. Documenting the Change for Others

Now that the new Redux pattern exists in a working way in the Tailwind codebase, it was important to cement the knowledge of how to use it in our codebase. We have an internal *Tailwind Engineering Book*, to which I added some code examples:

![Tailwind Book](https://user-images.githubusercontent.com/708562/65170800-6e3b6500-da17-11e9-8ec7-f145157dbd51.png)

By doing so, anyone on the team who wants to use the new Redux pattern has one place where they can look to find working examples in our codebase.

Since making the Emoji Picker change, I've added a few more working examples of the new Redux pattern to the codebase. Each time I did so, I put those examples in the *Tailwind Engineering Book*, so that it's a robust reference covering different Redux use-cases.

----

As you can see, it didn't take much actual code to implement the Redux change covered in [part one]() of this series. The process of making the change was much more about:

1. Determining whether the change made sense for Tailwind.
1. Getting buy-in from the team and soliciting alternative ideas.
1. Documenting the new pattern in such a way that others can use it in the future.

### Tailwind is Hiring!

At Tailwind, we're always working to improve our stack and use emerging technologies. If that sounds interesting to you, [join us!](https://www.tailwindapp.com/about)
