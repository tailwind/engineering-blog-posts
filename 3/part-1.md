# Improving an Existing React Codebase with Better State Management – Part 1

*This is part one of a three part series. Be sure to check out parts [two]() and [three]()!*

At Tailwind, we're quick to adopt new technologies that allow us to ship faster, safer, and cleaner code. While the company was founded more than six years ago, our tech stack looks very different today as compared to the early days.

Where new features were once made in jQuery, they're now built using React. In fact, Tailwind adopted React so early (2015) that we initially used [`hyperx`](https://github.com/choojs/hyperx) to do templating, because [`jsx`](https://reactjs.org/docs/introducing-jsx.html) had not yet become the de facto templating solution for React.

Our adoption of React allowed us to move much faster and build far more performant components than with jQuery. Overall, the adoption was a huge win. However, because React was a new technology at the time, we developed some code patterns that differ from what is commonly seen in React apps built today; particularly around state management / Redux. This pattern isn't wrong per se, but a couple of months ago, we considered whether or not it was a good idea to continue using it.

In investigating our React + Redux architecture and comparing it to the way that most React + Redux apps are built, I found that we're missing out on some of the benefits of doing React + Redux "by the book." In this post, you'll see the differences between Tailwind's React + Redux today and how it could be structured, if we were using the popular state management patterns available today.

This is the first post in a series of three about the process of adopting a new React + Redux approach in an existing codebase. You'll find a link to part two at the end of this post.

-----

## Tailwind Initial State vs. Common Redux Initial State
Here's how we initialize `state` in the Tailwind app:
```js
function initializeState (initialState) {
    return {
        profiles: {
            profilesById: {...}
        },
        posts: {
            postsById: {...}
        },
        ...
    }
}
```

One top-level object that contains all the various pieces of initial `state`. I simplified the example above, but in our actual codebase there are +40 top level state keys, most of which contain several levels of subkeys. The upshot is that our `state` initializer file is >1,000 lines of code!

Contrast this with the common Redux `state` initialization pattern:
```js
const initialProfilesState = {
    profileData: {
        profilesById: {...}
    }
}

const profilesReducer(state = initialProfilesState, action) {
    ...
}

const initialPostsState = {
    postsData: {
        postsById: {...}
    }
}

const postsReducer(state = initialPostsState, action) {
    ...
}

const rootReducer = combineReducers({
    profiles: profilesReducer,
    posts: postsReducer,
    ...
})
```

As you can see, in the examples above we make each reducer responsible for specifying its own initial `state`, and we combine reducers together to form our global store. This provides a few niceties versus our current implementation:
1. Initial `state` is more modular, rather than being defined in one large object.
2. Each reducer defines its own section of `state`, and more importantly, only has access to that section. The `postsReducer`, for example, doesn't have access to mutate `state.profiles`. We'll discuss why this matters in the next section.

## Tailwind Reducers vs. Common React + Redux Reducers & Actions
Here's how Tailwind's reducers look today:
```js
function getProfileReducer (state, params, ExternalServiceWrapper) {
    getProfileDataFromAPI(`https://api.tailwindapp.com/profiles/${params.profileId}`),
        (err, response) => {
            state.profiles.profilesById[params.profileId] = response.body;
        });

    return state;
}
```

There's a lot going on in this reducer. In fact, it's more than just a reducer. It's kind of a action-reducer-combo function. It first makes an API call (using a class we built called `ExternalServiceWrapper`), then it sets `state` using the result of that API call.

Commonly in React + Redux, you would see the same behavior split up into two functions like so:

```js
// Reducer:
export const profilesReducer(state = initialProfilesState, action) {
    switch (action.type) {
        case 'SET_PROFILE_DATA':
            state[action.profileId] = action.data;
            return state;
        default:
            return state
    }
}

// Action creator:
export const getProfileData = params => {
    return async dispatch => {
        const profileData = await getProfileDataFromAPI(params.profileId);
        dispatch({
            type     : 'SET_PROFILE_DATA',
            profileId: profileId,
            data     : profileData
        });
    }
}
```

The Redux pattern here provides some advantages over the Tailwind pattern:
1. Notice how the original `getProfileReducer` reducer has access to the full `state` object. In our example it's only modifying `state.profiles`, but it could make changes to any other area of Redux `state`. As we touched on in the previous section, the typical React + Redux pattern only allows reducers to change a small portion of `state` (in this example, `state.profiles`). This is the ideal encapsulation for a reducer, because it's much easier to see how a given area of state will change when there's just one reducer responsible for it. In otherwords, reducers are much easier to audit.
2. In the common React + Redux setup, reducers are simple, predictable, static functions. They can be thoroughly tested without having to think about API calls, which speeds up the development workflow.

## Tailwind Components vs. Common React + Redux Components
Tailwind components will typically receive the entire `state` object as a prop:
```js
export default class ProfileDetails extends Component {
    ...
    render() {
        const { state, userId } = this.props;
        return (
            <div className='greeting'>
                Hello, {state.profiles.profilesById[userId].firstName}!
            </div>
            ...
        )
    }
}
```

This pattern has several problems:
- Components that are directly tied to a `state` object are less re-usable. Ideally, components should have a limited scope and should be designed that they can be reused as needed. Passing around very large objects, like our `state`, is usually a smell that a component is not as modular as it could be.
- It can cause unneeded re-renderings. If `state.posts` changes, for example, `ProfileDetails` will re-render even though `state.posts` isn't used anywhere in the component. The performance impact of these extra render cycles isn't usually noticeable, but in very large components it can cause performance issues and can be a headache to manage.
- It's unnecessary; the component above doesn't need all of `state`.
- To test the `ProfileDetails` component, we now have to mock state. This shouldn't be needed. A component should only care about the props that are passed to it, its own local state, and the things it needs to render. It would be great if we could keep `ProfileDetails` unaware of global state.
- This pattern just looks weird! We have to manage both local component state (`this.state`) in React components, and we have `state` as a prop (`this.props.state`).

The common React + Redux pattern is a big improvement because it fixes all of the problems listed above:
```js
class ProfileDetails extends Component {
    ...
    render() {
        const { profile } = this.props;
        return (
            <div className='greeting'>
                Hello, {profile.firstName}!
            </div>
            ...
        )
    }
}

const mapStateToProps = (state, ownProps) => ({
    profile: state.profiles.profilesById[ownProps.userId]
});

export default connect(mapStateToProps)(ProfileDetails)
```

This method fixes all of the problems listed above:
- No unneeded re-renderings at the component level. If `state.posts` changes, `ProfileDetails` isn't aware of the change and doesn't need to re-render because of it.
- We're not providing `ProfileDetails` with unnecessary data by passing in all of `state`. We only pass in what we need via `mapStateToProps`.
- It's easy to see what parts of `state` are being passed to `ProfileDetails` by looking at `mapStateToProps`. Even better, since `mapStateToProps` is a simple, static function, we can test it easily without doing any kind of component mounting or store mocking. Brilliant!
- We only have one concept of `state` inside the component: `this.state`.

## To Refactor or not to Refactor, That is the Question

Summing up the main differences between Tailwind's React + Redux and the standard React + Redux pattern:

#### Initial state

❌ *Tailwind pattern:* One large object that defines all initial `state` for the app.

✅ *Standard pattern:* Each reducer defines its own initial `state`, and those reducers are used to create the initial `state` for the app with `combineReducers()`.

#### Reducers

❌ *Tailwind pattern:* Use reducers to both change `state` and interact with the outside world.

✅ *Standard pattern:* Limit API calls to actions and `state` changes to reducers.

#### Components

❌ *Tailwind pattern:* Pass the full `state` object to components and use it as needed.

✅ *Standard pattern:* Extract needed properties from `state` and pass them explicitly to components using `connect()`

---------

We can see that between these different methods, the modern day React + Redux way of doing things is preferable. It passes the "magic wand" test, meaning that if we could wave a magic wand and have Tailwind's app work this way right now, we would.

Unfortunately, we don't have a magic wand handy, and refactoring the entire app to use this new `state` management pattern is a dangerous undertaking! For context, Tailwind has:
- 100+ components that receive `state` as a prop
- 250+ reducers
- 40+ top-level keys in `state`

All of which would need to be migrated in order for Tailwind to fully switch over. This would take a tremendous teamwide effort, which is probably not feasible.

What might be worthwhile, however, is building *future* components and reducers in this way. If Tailwind could improve our development speed, testing, and performance for new features, that'd be a huge win for team.

## So What's Next?
After going through the process outlined above, it was clear to me that moving to a new pattern of state management for would be a good solution for the team.

From there, my goal was to present this idea to the Tailwind engineering team for feedback, which I cover in part two of this series [TODO LINK TO PART TWO].
