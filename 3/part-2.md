# Improving an Existing React Codebase with Better State Management – Part 2

*This is part two of a three part series. Be sure to check out parts [one]() and [three]()! [<-- TODO fill out these links]*

In the first post of this series, we analyzed the Tailwind codebase, in particular we looked at the way that we manage React + Redux state at Tailwind. We compared Tailwind's state management pattern with the common statement pattern seen today. The results were conclusive: Tailwind's infrastructure would be improved by using the React + Redux "by the book" approach.

However, there are two problems with adopting a new way of state management in Tailwind's codebase:
1. Tailwind has hundreds of reducers and components that would need migrating.
1. Adopting a new way of doing things requires a change in team thinking and a willingness to change current habits with the belief that those changes will work well in the long run.

The solution to problem #1 is not to migrate hundreds of individual files. That would be a herculean effort, and as the saying goes: *"If it ain't broke, don't fix it."*

The solution to this problem is not to overhaul an entire codebase to use a new state management pattern, but rather to build *new* features using this pattern. That way, the codebase increasingly uses the new pattern over time, but without risk of breaking features or causing headaches from trying to do too much too quickly.

The solution to Problem #1 is straightforward. Problem #2, on the other hand, has a more nuanced answer, and it's the subject of this post.

As a developer working on a team, how do you affect teamwide habit change? How do you get buy-in that we, as a team, can build things in a better way, when the current pattern works well? This process was a good experience for me in answering these questions. The main steps for proposing this change included:

1. **Starting a conversation with the team**
1. **Proposing the idea formally**
1. **Providing team members with a platform to suggest alternatives**

Let's dig into these steps in depth.

----

### 1. Starting a conversation with the team

The process of getting buy-in from the management perspective was straightforward. Andres (Tailwind's engineering manager) and I took a couple of hours to review the codebase and look at examples of our approach versus the *by-the-book* approach, after which, Andres was sold on the idea of migrating to a new way of managing Redux. From there, we discussed how best to get the team involved in the discussion and solicit feedback from them.

First, I broadcasted the idea to the team in a slack post + internal *Request for Comments (RFC)* in April. This was the first time for most of the engineering team to consider this idea. From there, I directly tagged a couple of our React gurus and asked them to review the idea and provide feedback.

![Redux RFC in Tailwind's Engineering Book](https://user-images.githubusercontent.com/708562/65162200-e2b9d800-da06-11e9-8781-3594a9e02443.png)

The general reception to the idea was positive and the few people who had the chance to dig in on it seemed to generally agree that it could be a good improvement in our codebase. From there, I knew that the idea was validated enough to present to the larger group.

### 2. Proposing the idea formally

Once a few people had a chance to explore the concept and provide initial feedback, the next step was to present the idea to the team formally. This was done via a Google Slides presentation.

![Redux Presentation to the Engineering Team](https://user-images.githubusercontent.com/708562/65162856-eef26500-da07-11e9-8864-a463608ee65d.png)

In the presentation, I walked the engineering team through:
1. **The motivation for this proposal –** That we can improve our Redux code patterns to be more testable, more scalable, and more performant
1. **The status quo –** How we manage Redux state today
1. **The potential future –** How we could manage Redux state tomorrow
1. **Tradeoffs –** Pros and cons of the *status quo* vs. the *potential future* approach

The middle sections included a bunch of code examples, highlighting specific issues with our codebase as well as specific improvements from the new code patterns.

Once I finished the presentation, we had a candid discussion for about 20 minutes where people had the chance to ask good questions – for example: "What performance issues do we have today and how do this address them?"

Tailwind's React gurus, who had reviewed the *Request for Comments*, were able to chime in and help to answer some of these questions, being familiar with Tailwind's React codebase and being informed on this approach from having already reviewed the RFC. In that way, there was some buy-in from the team before coming into the presentation.

### 3. Providing team members with a platform to suggest alternatives

After the presentation, the entire engineering team had been exposed to the idea and had been given a chance to ask questions and present concern. The last remaining step was to prompt the team to answer the question **"Should we adopt this approach?"**

Up to this point, no one on the team had voiced any dissent towards the idea of making a change to our state management, but no one had been given a chance to present alternative ideas either.

To give a final chance for others to express alternative solutions, I dropped a short post in our `#engineering` Slack channel, proposing that others suggest alternatives, such as:

1. Doing nothing
1. Using an alternative React pattern (e.g. React [Hooks](https://reactjs.org/docs/hooks-intro.html))
1. Implementing a partial pattern change (for example, using `connect` but no reducer changes)

The engineering team by and large felt that alternatives were interesting but not necessarily better. No one championed alternative ideas or the status quo.

----

Ultimately, after the process of socializing the idea to the engineering team here at Tailwind, we felt that the team bought into the idea and it was time to put it in action. How did we do so? For that, you'll have to read [part 3]() [<-- TODO fill this link out] of this series.
