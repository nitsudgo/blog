---
slug: programming/Notion-As-A-CMS-For-NextJS-Blogging
created_at: 2023-11-12 19:05
is_template: "0"
featured_image: 
title: Notion as a CMS for NextJS Blogging
---
*I’ve been looking for a simple way to integrate a blog and rudimentary content management system into my Next.js web application. After playing around with a few options such as scraping from a separate Jekyll static blog, I finally discovered, and settled on using Notion and its Publish to Web feature. By combining the flexibility and ease of writing in Notion with the `react-notion-x` package, I can finally make and manage my content without worrying too much about publishing and post CRUD infrastructure.*

> This is an old post migrated from ancient times. Whilst the content should still be okay, a lot of things have changed since then. For one, I don't use Notion as a CMS anymore!
# Introduction

In case you haven't already noticed, the entire blog portion of this website (i.e. the blog post listing page, and the blog posts themselves) has been crafted in [Notion](https://www.notion.so/). I first heard of Notion when some colleagues brought it up briefly during a work discussion but I didn't think much of it at the time. At some point during my efforts to improve self productivity, I decided to take a closer look at it to see if it could boost my efficiency and organizational planning skills. I was quite impressed at how intuitive Notion was and how easy it was to get started creating things like Kanban boards, habit trackers, write-ups, documentation, and more.

One thing that really stood out for me was the ability to publish my created Notion pages to the web. I thought that if it really *was* that easy to make pages and share them online publicly, Notion would make a great candidate for a simple yet flexible blogging implementation- something that I had been trying to do manually up until then ☠️

# The original approach

I won't talk too much about the previous methods I used to build the blog section, but in a nutshell, when I first created this portfolio app I had originally intended for the blog portion to be constructed with [Jekyll](https://jekyllrb.com/), and then hosted on [GitHub Pages](https://pages.github.com/). Then, I would use a library like [axios](https://github.com/axios/axios) to send a request to the relevant pages and serve the blog posts by inserting the resultant HTML into a container.

This approach left a lot to be desired- although it would have given me complete control of page layout, behaviour, and CI/CD of the blog, I found the approach to be cumbersome and quite involved for something that should have been really simple. I also didn't need such fine control for blogging, and I honestly preferred an implementation where I could focus more on actually *writing* the post instead of worrying about *how* it gets posted.

Not to mention it's way more fun to write things up using the intuitive Notion features, as opposed to slogging it out in a boring old IDE. I mean I love VS Code as much as the next person but sometimes you just need a change of scenery you know?

![https://media.giphy.com/media/NQhZRrDaUuVYOpg6GI/giphy.gif](https://media.giphy.com/media/NQhZRrDaUuVYOpg6GI/giphy.gif)

# The Notion approach

A quick google search led me to a React library called [react-notion-x](https://github.com/NotionX/react-notion-x). This library would do most of the heavy lifting and allow me to concentrate in writing up pages in Notion itself. I can then fetch and render them as React components on my app!

> 🤫 In the spirit of DRY (Don't Repeat Yourself), I won't be going through the entire implementation process step by step, as this has already been well documented. Instead, I'll be outlining the important bits and considerations that I came across when working.

**Before we start, it should be noted that the pages you wish to render should be published to the web. You can do this by opening the Notion page you wish to publish, and activating Share → Share to Web. If you don't do this, you'll have to get an access token and pass that to the component we'll be using. I'm not gonna cover that in this writeup though.**

## Rendering a single Notion page

The implementation behind rendering a single Notion page is pretty trivial, and the entire code of the blog listing page (`/blog`) is given below:

```jsx

// Next
import Link from 'next/link';

// Notion
import { NotionAPI } from 'notion-client'
import { Collection, CollectionRow, NotionRenderer } from 'react-notion-x'

const notion = new NotionAPI()
export default function BlogHome ({ recordMap, darkMode }) {
	
	return (
		<div className='text-left overflow-hidden'>
		{/* Use mapPageUrl here to prepend /blog to any URLs from the Notion page
		This will mean every path from notion will now have /blog/ at the start */}
		<NotionRenderer
			mapPageUrl={(pageUrl) => `/blog/${pageUrl}`}
			recordMap={recordMap}
			fullPage={true}
			darkMode={darkMode}
			components={{
			collection: Collection,
			collectionRow: CollectionRow,
			pageLink: ({
			href,
			as,
			passHref,
			prefetch,
			replace,
			scroll,
			shallow,
			locale,
			...props
		}) => (
			<Link
				href={href}
				as={as}
				passHref={passHref}
				prefetch={prefetch}
				replace={replace}
				scroll={scroll}
				shallow={shallow}
				locale={locale}
			>
				<a {...props} />
			</Link>
		),
		}} />
		</div>
	);
}
	
export async function getStaticProps(_context) {
	const recordMap = await notion.getPage(process.env.NOTION_ROOT_POST_ID);
	
	return {
		props: {
			recordMap: recordMap
		},
		revalidate: 10
	}
}
```

First, we import the required packages:

- `NotionAPI` is used to interface with the Notion APIs. Note that after importing, I create a new instance with `const notion = new NotionAPI()`.

- `Collection` and `CollectionRow` are components that give us the functionalities of Collections in Notion. They are kept separate for space-saving reasons as they are relatively heavy, but since my blog index relies on using a collection to list the posts, I’ll need to import them.

- `NotionRenderer` is the component that will render the actual page out for us!

Then, before the page loads, I use the Next.js data fetching method `getStaticProps` to fetch the page I want to render from Notion and pass this page straight to the props of the BlogHome page component.

I use the `getPage` function for this and pass it the ID of the page I want to fetch. In this case, I have stored that in the environment variable `NOTION_ROOT_POST_ID`. To get the IDs of your own posts, simply copy the last alphanumeric chunk of text from the URL of the published page. For example:

If after publishing, your URL is:

[`https://motley-phlox-570.notion.site/Notion-as-a-CMS-for-Next-js-blogging-a0670f656ab7477bb5598a0643cece62`](https://www.notion.so/Notion-as-a-CMS-for-Next-js-blogging-a0670f656ab7477bb5598a0643cece62?pvs=21)

Then your page ID would be `a0670f656ab7477bb5598a0643cece62`. Easy! 😊

<div style="width:100%;height:0;padding-bottom:75%;position:relative;"><iframe src="https://giphy.com/embed/aNbGyHcDYphNbhe4EE" width="100%" height="100%" style="position:absolute" frameBorder="0" class="giphy-embed" allowFullScreen></iframe></div><p><a href="https://giphy.com/gifs/achievementhunter-rooster-teeth-achievement-hunter-roosterteeth-aNbGyHcDYphNbhe4EE">via GIPHY</a></p>

In the page component itself, I use the `NotionRenderer` component to render my page! Notice the following props I pass to it:

- **mapPageUrl** *(url) ⇒ string*

- When you click on links within the rendered Notion page that go to other Notion pages, the new page ID will be appended onto your current URL. This prop allows us to alter the format of the new URL. In my case, the new page IDs from this page correspond to blog posts! I choose to prepend the new page IDs with `/blog/` so that the Next.js routing can pick it up and I can render the blog post from there 😎

- **recordMap** *Object*

- The result of the `NotionRenderer.getPage()` function call. Think of it like a blueprint of the Notion page that tells the component how to render it!

- **fullPage** *Boolean*

- Whether or not to render the page icon and header image, if any.

- **darkMode** *Boolean*

- Whether or not the page should render in Notion dark mode.

- **components** *Object*

- We use this prop to pass in components and overrides to the `NotionRenderer`. Notice that I pass in `Collection` and `CollectionRow`, which give the component the ability to render Notion collections. I also pass in a function to override `pageLink` . When I do this, I am converting the Notion links that are being rendered into Next.js `Link` components, which makes intra-site navigation much smoother and easier to deal with!

With these concepts in mind, you should be able to render any Notion page you like! Updating the content of the page is as simple as editing the page directly in Notion; your changes will take effect almost instantaneously.

<div style="width:100%;height:0;padding-bottom:56%;position:relative;"><iframe src="https://giphy.com/embed/349qKnoIBHK1i" width="100%" height="100%" style="position:absolute" frameBorder="0" class="giphy-embed" allowFullScreen></iframe></div><p><a href="https://giphy.com/gifs/reaction-program-programmers-349qKnoIBHK1i">via GIPHY</a></p>

# Closing Thoughts

Notion has proven itself to be an invaluable tool in one's productivity arsenal, and now we see that it can function quite reliably as a CMS and blog! Title has been only fortified due to the ability to render its pages as React components, giving us a great balance between the ease of implementation, and the speed and flexibility of the content we can create. There is much that I haven't yet touched on in terms of Notion rendering in React such as the use of Collections (Notion Databases), more exciting layouts, and more exotic embeddings / integrations. I hope to one day explore them all!

# Useful Links

- [GitHub - NotionX/react-notion-x: Fast and accurate React renderer for Notion. TS batteries included. ⚡️](https://github.com/NotionX/react-notion-x)
- [Notion Kit Test Suite](https://react-notion-x-demo.transitivebullsh.it/)
- [GitHub - transitive-bullshit/nextjs-notion-starter-kit: Deploy your own Notion-powered website in minutes with Next.js and Vercel.](https://github.com/transitive-bullshit/nextjs-notion-starter-kit)