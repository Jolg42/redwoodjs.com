# Welcome to Redwood, Part II: Redwood's Revenge

Part 1 of the tutorial was a huge success! It introduced hundreds (maybe thousands?)
of developers to what Redwood could do to make web development in the Javascript
ecosystem a delight. But that was just the beginning.

If you read the README [closely](https://github.com/redwoodjs/redwood#technologies)
you may have seen a few technologies that we didn't
touch on at all in the first tutorial: [Storybook](https://storybook.js.org/) and [Jest](https://jestjs.io/). In reality, these
have been core to the very idea of Redwood from the beginning—an improvement to the
entire experience of developing a web application.

While they're totally optional, we believe using these two tools will greatly
improve your development experience, making your applications easier to develop,
easier to maintain, and easier to share with a larger team. In this second tutorial
we're going to show you how.

Oh, and while we're at we'll introduce Role-based Authorization Control (RBAC),
which wasn't available when we wrote the first tutorial, but is now, and it's amazing.

## Prerequisites

We highly recommend going through the first tutorial first, or at least have built
a slightly complex Redwood app on your own. You've hopefully got experience with:

* Authorization
* Cells
* GraphQL & SDLs
* Services

If you haven't been through the first tutorial, or maybe you went through it on an
older version of Redwood (before 0.19.0) you can clone this repo which contains
everything built in part 1 and also adds a little styling so it isn't quite so...ugly.
Don't get us wrong, what we built in part 1 had a great personality! We just gave it
some hipper clothes and a nice haircut. We used [TailwindCSS](https://tailwindcss.com)
to style things up and added a `<div>` or two to give us some additional hooks to hang
some styling.

    git clone https://github.com/redwoodjs/redwood-tutorial
    cd redwood-tutorial
    yarn install
    yarn rw db up
    yarn rw db seed
    yarn dev

That'll check out the repo, install all the dependencies, create your local database
and fill it with a few blog posts, and finally start up the dev server. Your browser
should open to a fresh new blog app:

![image](https://user-images.githubusercontent.com/300/95521547-a5f8b000-097e-11eb-911c-5fde4bed6d97.png)

## Introduction to Storybook

Let's see what this Storybook thing is all about. Run this command to start up the Storybook server:

    yarn rw storybook

After some compling you should get a message saying that Storybook has started and it's
available at http://localhost:7910

![image](https://user-images.githubusercontent.com/300/95522673-8f078d00-0981-11eb-9551-0a211c726802.png)

If you poke around at the file tree on the left you'll see all of the components, cells,
layouts and pages we created during the tutorial. Where did they come from? You may recall
that everytime we generated a new page/cell/component we actaully created at least *three* files:

* BlogPost.js
* BlogPost.stories.js
* BlogPost.test.js

> If you generated a cell then you also got a `.mock.js` file (more on those later).

Those `.stories.js` files are what makes the tree on the left side of the Storybook browser
possible! From their homepage, Storybook describes itself as:

*"Storybook is an open source tool for developing UI components in isolation for React, Vue, Angular, and more. It makes building stunning UIs organized and efficient."*

So, the idea here is that you can build out your components/cells/pages in isolation, get them
looking the way you want and displaying the correct data, then plug them into your full application.

When Storybook opened it should have opened Components > BlogPost > Generated which is the
generated component we created to display a single blog post. If you open `web/src/components/BlogPost/BlogPost.stories.js` you'll see what it takes to explain this component to Storybook, and it isn't much:

```javascript
import BlogPost from './BlogPost'

export const generated = () => {
  return (
    <BlogPost
      post={{
        id: 1,
        title: 'First Post',
        body: 'Neutra tacos hot chicken prism raw denim...',
      }}
    />
  )
}

export default { title: 'Components/BlogPost' }
```

You import the component you want to use and then all of the named exports in the file will be
a single "story" as displayed in Storybook. In this case the generator named it "generated"
which shows as the "Generated" story in the tree view:

```terminal
Components
└── BlogPost
    └── Generated
```

This makes it easy to create variants of your component and have them all displayed together.

> Where did that sample blog post data come from? We (the Redwood team) added that to the story in the `redwood-tutorial` repo to show you what a story might look like after you hook up some sample data. The rest of the tutorial will be showing you how to do this yourself with new components as you create them.

## Our First Story

Let's say that on our homepage we only want to show the first couple of sentences in our blog post and you'll have to click through to see the full post.

First let's update the `BlogPost` component to contain that functionality:

```javascript{5-7,9,18}
// web/src/components/BlogPost/BlogPost.js

import { Link, routes } from '@redwoodjs/router'

const truncate = (text, length) => {
  return text.substring(0, length) + '...'
}

const BlogPost = ({ post, summary = false }) => {
  return (
    <article className="mt-10">
      <header>
        <h2 className="text-xl text-blue-700 font-semibold">
          <Link to={routes.blogPost({ id: post.id })}>{post.title}</Link>
        </h2>
      </header>
      <div className="mt-2 text-gray-900 font-light">
        {summary ? truncate(post.body, 100) : post.body}
      </div>
    </article>
  )
}

export default BlogPost
```

We'll pass an additional `summary` prop to the component to let it know if should show
just the summary or the whole thing. We default it to `false` to preserve the existing
behavior (always showing the full body).

Now in the story, let's create a `summary` story that uses `BlogPost` the same way that
`generated` does, but adds the new prop. We'll take the content of the sample post and
put that in a constant that both stories will use. We'll also rename `generated` to
`full` to make it clear what's different between the two:

```javascript{5-14,16-18,20-22}
// web/components/BlogPost/BlogPost.stories.js

import BlogPost from './BlogPost'

const POST = {
  id: 1,
  title: 'First Post',
  body: `Neutra tacos hot chicken prism raw denim, put a bird on it enamel pin
         post-ironic vape cred DIY. Street art next level umami squid. Hammock
         hexagon glossier 8-bit banjo. Neutra la croix mixtape echo park four
         loko semiotics kitsch forage chambray. Semiotics salvia selfies jianbing
         hella shaman. Letterpress helvetica vaporware cronut, shaman butcher
         YOLO poke fixie hoodie gentrify woke heirloom.`,
}

export const full = () => {
  return <BlogPost post={POST} />
}

export const summary = () => {
  return <BlogPost post={POST} summary={true} />
}

export default { title: 'Components/BlogPost' }
```

As soon as you save the change the stories Storybook should refresh and show the updates:

![image](https://user-images.githubusercontent.com/300/95523957-ed823a80-0984-11eb-9572-31f1c249cb6b.png)

Great! Now to complete the picture let's use the summary in our home page display of blog posts.
The actual Home page isn't what references the `BlogPost` component though, that's in the
`BlogPostsCell`. We'll add the summary prop and then check the result in Storybook:

```javascript{26}
// web/src/components/BlogPostsCell/BlogPostsCell.js

import BlogPost from 'src/components/BlogPost'

export const QUERY = gql`
  query BlogPostsQuery {
    posts {
      id
      title
      body
      createdAt
    }
  }
`

export const Loading = () => <div>Loading...</div>

export const Empty = () => <div>Empty</div>

export const Failure = ({ error }) => <div>Error: {error.message}</div>

export const Success = ({ posts }) => {
  return (
    <div className="-mt-12">
      {posts.map((post) => (
        <BlogPost key={post.id} post={post} summary={true} />
      ))}
    </div>
  )
}
```

![image](https://user-images.githubusercontent.com/300/95525432-f4ab4780-0988-11eb-9e9b-8df6641452ec.png)

And if you head to the real site you'll see the summary there as well:

![image](https://user-images.githubusercontent.com/300/95527363-ef9ac800-0989-11eb-9c53-6dc8ab58799c.png)

Storybook makes it easy to create your components in isolation and actually helps
enforce a general best practice when building React applications: components should
be self-contained and reusable by just changing the props that are sent in.

## Building a Component with Storybook

What's our blog missing? Comments. Let's add a simple comment engine so people can leave
their completely rational, well-reasoned comments on our blog posts. It's the Internet,
what could go wrong?

There are a couple of ways we could go about building this new feature:

1. Start with the form, then the comment display
2. Start with the comment display, then add the form

To keep things simple let's start with the display first, then we'll move on to more complex work of a form and service to save data.

Let's create a component for the display of a single comment. First up, the generator:

```terminal
yarn rw g component Comment
```

Storybook should refresh and our "Generated" Comment story will be ready to go:

![image](https://user-images.githubusercontent.com/300/95784041-e9596400-0c87-11eb-9b9f-016e0264e0e1.png)

Let's think about what we want to ask users for and then display in a comment. How about just their name and the content of the comment itself? And we'll throw in the date/time it was created. Let's update the Comment component to accept a `comment` object with those two properties:

```javascript{3,6-7}
// web/src/components/Comment/Comment.js

const Comment = ({ comment }) => {
  return (
    <div>
      <h2>{comment.name}</h2>
      <time datetime={comment.createdAt}>{comment.createdAt}</time>
      <p>{comment.body}</p>
    </div>
  )
}

export default Comment
```

Once you save that file and Storybook reloads you'll see it blow up:

![image](https://user-images.githubusercontent.com/300/95784285-6684d900-0c88-11eb-9380-743079870147.png)

We need to update the story to include that comment object:

```javascript{8-11}
// web/src/components/Comment/Comment.stories.js

import Comment from './Comment'

export const generated = () => {
  return (
    <Comment
      comment={{
        name: 'Rob Cameron',
        body: 'This is the first comment!',
        createdAt: '2020-01-01T12:34:56Z'
      }}
    />
  )
}

export default { title: 'Components/Comment' }
```

> Note that Datetimes will come from GraphQL in ISO8601 format

Storybook will reload and be much happier:

![image](https://user-images.githubusercontent.com/300/95785006-ccbe2b80-0c89-11eb-8d3b-bdf5ad5a6d63.png)

Let's add a little bit of styling and date conversion to get this Comment component looking like a nice, completed design element:

```javascript{3-7,11-18}
// web/src/components/Comment/Comment.js

const formattedDate = (datetime) => {
  const parsedDate = new Date(datetime)
  const month = parsedDate.toLocaleString('default', { month: 'long' })
  return `${parsedDate.getDate()} ${month} ${parsedDate.getFullYear()}`
}

const Comment = ({ comment }) => {
  return (
    <div className="bg-gray-200 p-8 rounded-lg">
      <header className="flex justify-between">
        <h2 className="font-semibold text-gray-700">{comment.name}</h2>
        <time className="text-xs text-gray-500" dateTime={comment.createdAt}>
          {formattedDate(comment.createdAt)}
        </time>
      </header>
      <p className="text-sm mt-2">{comment.body}</p>
    </div>
  )
}

export default Comment
```

![image](https://user-images.githubusercontent.com/300/95786526-9afa9400-0c8c-11eb-9d75-27c996ca018a.png)

It's tough to see our rounded corners, but rather than adding margin or padding to the component itself (which would add them everywhere we use the component) let's add a margin in the story so it only shows in Storybook:

```javascript{7,15}
// web/src/components/Comment/Comment.stories.js

import Comment from './Comment'

export const generated = () => {
  return (
    <div className="m-4">
      <Comment
        comment={{
          name: 'Rob Cameron',
          body: 'This is the first comment!',
          createdAt: '2020-01-01T12:34:56Z',
        }}
      />
    </div>
  )
}

export default { title: 'Components/Comment' }
```

> A best practice to keep in mind when designing in HTML and CSS is to keep a visual element responsible for its own display only, and not assume what it will be contained within. In this case a Comment doesn't and shouldn't know where it will be displayed, so it shouldn't add any design influence *outside* of its container (like forcing a margin around itself).

Now we can see our roundedness quite easily in Storybook:

![image](https://user-images.githubusercontent.com/300/95786006-aac5a880-0c8b-11eb-86d5-105a3b929347.png)

> If you haven't used TailwindCSS before just know that the `m` in the className is short for "margin" and the `4` refers to four "units" of margin. By default one unit is 0.25rem. So "m-4" is equivalent to `margin: 1rem`.

Our amazing blog posts will obviously garner a huge and passionate fanbase and we will very rarely have only a single comment. Let's work on displaying a list of comments.

## Multiple Comments

Let's think about where our comments are being displayed. Probably not on the homepage, since that only shows a summary of each post. A user would need to go to the full page to show the comments for that blog post. But that page is only fetching the data for the single blog post itself, nothing else. We'll need to get the comments and since we'll be fetching *and* displaying them, that sounds like a job for a Cell.

> **Couldn't the query for the blog post page also fetch the comments?**
>
> Yes, it could! But the idea behind Cells is to make components even more [composable](https://en.wikipedia.org/wiki/Composability) by having them be responsible for their own data fetching *and* display. If we rely on a blog post to fetch the comments then the new Comments component we're about to create now requires something else to fetch the comments and pass them in. If we re-use the Comments component somewhere, now we're fetching comments in two different places.
>
> **But what about the Comment component we just made, why doesn't that fetch its own data?**
>
> There aren't any instances I (the author) could think of where we would ever want to display only a single comment in isolation—it would always be a list of all comments on a post. If displaying a single Comment was common for your use case then it could definitely be converted to a CommentCell and have it responsible for pulling the data for that single comment itself. But keep in mind that if you have 50 comments on a blog post, that's now 50 GraphQL calls that need to go out, one for each comment. There's always a tradeoff!
>
> **Then why make a standalone Comment component at all? Why not just do all the display in the CommentsCell?**
>
> We're trying to start in small chunks to make the tutorial more digestable for a new audience so we're starting simple and getting more complex as we go. But it also just feels *nice* to build up a UI from these smaller chunks that are easier to reason about and keep separate in my (the author's) head.
>
> **But what about—**
>
> Look, we gotta end this sidebar and get back to building this thing. You can ask more questions later, promise!

Let's generate a `CommentsCell`:

```terminal
yarn rw g cell Comments
```

Storybook updates with a new **CommentsCell** under the **Cells** folder. Let's update the Success story to use the Comment component created earlier:

```javascript{3,20}
// web/src/components/CommentsCell/CommentsCell.js

import Comment from 'src/components/Comment'

export const QUERY = gql`
  query CommentsQuery {
    comments {
      id
    }
  }
`

export const Loading = () => <div>Loading...</div>

export const Empty = () => <div>Empty</div>

export const Failure = ({ error }) => <div>Error: {error.message}</div>

export const Success = ({ comments }) => {
  return comments.map((comment, i) => <Comment key={i} comment={comment} />)
}
```

Note that we're also passing a `key` prop to make React happy.

If you check Storybook, you'll seen an error. We'll need to update the `mock.js` file that came along for the ride when we generated the Cell so that it returns an array instead of just a simple object with some sample data:

```javascript{4-11}
// web/src/components/CommentsCell/CommentsCell.mock.js

export const standard = (/* vars, { ctx, req } */) => ({
  comments: [
    {
      name: 'Rob Cameron', body: 'First comment', createdAt: '2020-01-02T12:34:56Z'
    },
    {
      name: 'David Price', body: 'Second comment', createdAt: '2020-02-03T23:00:00Z'
    },
  ]
})

```

> What's this `standard` thing? Think of it as the standard, default mock if you don't do anything else. We would have loved to use the name "default" but that's already a reserved word in Javascript!

Storybook refreshes and we've got comments! We've got the same issue here where it's hard to see our rounded corners and also the two separate comments are are hard to distinguish because they're right next to each other:

![image](https://user-images.githubusercontent.com/300/95799544-dce60300-0ca9-11eb-9520-a1aac4ec46e6.png)

The gap between the two comments *is* a concern for this component, since it's responsible for drawing multiple comments and their layout. So let's fix that in CommentsCell:

```javascript
export const Success = ({ comments }) => {
  return (
    <div className="-mt-8">
      {comments.map((comment, i) => (
        <div key={i} className="mt-8">
          <Comment comment={comment} />
        </div>
      ))}
    </div>
  )
}
```

We had to move the `key` prop to the surrounding `<div>`. We then gave each comment a top margin and removed an equal top margin from the entire container to set it back to zero.

Let's add a margin around the story itself, similar to what we did in the Comment story:

```javascript
export const success = () => {
  return Success ? (
    <div className="m-8 mt-16">
      <Success {...standard()} />
    </div>
  ) : null
}
```

> Why both `m-8` and `mt-16`? One of the fun rules of CSS is that if a parent and child both have margins, but no border or padding between them, their `margin-top` and `margin-bottom` [collapses](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Box_Model/Mastering_margin_collapsing). So even though the story container will have a margin of 8 (which equals 2rem) remember that the container for CommentsCell has a -8 margin. Those two collapse and essential cancel each other out to 0 top margin. Setting `mt-16` sets a 4rem margin, which leaves us with 2rem, which is what we wanted to start with!

![image](https://user-images.githubusercontent.com/300/95800481-4cf58880-0cac-11eb-9457-ff3f1f0d34b8.png)

Looking good! Let's add our CommentsCell to the actual blog post display page:

```javascript{4,21}
// web/src/components/BlogPost/BlogPost.js

import { Link, routes } from '@redwoodjs/router'
import CommentsCell from 'src/components/CommentsCell'

const truncate = (text, length) => {
  return text.substring(0, length) + '...'
}

const BlogPost = ({ post, summary = false }) => {
  return (
    <article className="mt-10">
      <header>
        <h2 className="text-xl text-blue-700 font-semibold">
          <Link to={routes.blogPost({ id: post.id })}>{post.title}</Link>
        </h2>
      </header>
      <div className="mt-2 text-gray-900 font-light">
        {summary ? truncate(post.body, 100) : post.body}
      </div>
      {!summary && <CommentsCell />}
    </article>
  )
}

export default BlogPost
```

If we are *not* showing the summary, then we'll show the comments.

<!--
  TODO This may not be an issue after Peter gets a chance to look at mocks
  https://github.com/redwoodjs/redwood/issues/1374
-->

If you check out Components > BlogPost > Summary you'll see nothing has changed. But going to Components > BlogPost > Full and you'll see that Storybook doesn't like something:

![image](https://user-images.githubusercontent.com/300/95800778-18ce9780-0cad-11eb-927d-d308d2d02071.png)

What's happening here is that although the CommentsCell story uses the neighboring `mock.js` file, as soon as we use that component in another story, Storybook is just trying to use it like a regular component—it doesn't know about the `mock.js` file, that's a Redwood convention. Let's tell the `BlogPost` to use the data from `CommentsCell.mock.js`:

```javascript{4,18-20}
// web/src/components/BlogPost/BlogPost.stories.js

import BlogPost from './BlogPost'
import { standard as comments } from 'src/components/CommentsCell/CommentsCell.mock'

const POST = {
  id: 1,
  title: 'First Post',
  body: `Neutra tacos hot chicken prism raw denim, put a bird on it enamel pin
         post-ironic vape cred DIY. Street art next level umami squid. Hammock
         hexagon glossier 8-bit banjo. Neutra la croix mixtape echo park four
         loko semiotics kitsch forage chambray. Semiotics salvia selfies jianbing
         hella shaman. Letterpress helvetica vaporware cronut, shaman butcher
         YOLO poke fixie hoodie gentrify woke heirloom.`,
}

export const full = () => {
  mockGraphQLQuery('CommentsQuery', () => comments())

  return <BlogPost post={POST} />
}

export const summary = () => {
  return <BlogPost post={POST} summary={true} />
}

export default { title: 'Components/BlogPost' }
```

So, first we import the `standard` mock from `CommentsCell.mock.js` and rename it `comments` for clarity. Note that this is a function, not an object.

Next we use Redwood's built-in `mockGraphQLQuery()` function to return a fake response when a query named `CommentsQuery` is executed (that's the name of the query in `CommentsCell` main `QUERY`):

```javascript{4}
// web/components/CommentsCell/CommentsCell.js

export const QUERY = gql`
  query CommentsQuery {
    comments {
      id
    }
  }
`
```

When that query is found, return `comments()` (the mock data function we imported at the top).