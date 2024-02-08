{
    "title": "Rewriting my blog",
    "date": "2024-02-09"
}
****
On the 31st of January 2024 I started rewriting my blog. The previous version used handrolled HTML and CSS and was not very extensible. For every new post, I would have to manually create and edit pages to make it work. It also didn't work on mobile at all. That needed to change.

Off I went browsing the interwebs for a web framework that would allow me to write my blog posts in Markdown and wasn't too hard to think around. It needed to be small and not too batteries included. I'd touched Hugo before, but there was a lot of stuff in the project I used it on that made me dislike it. It does a lot of magic and it isn't very clear where what goes. Granted, we were using a website template, but I don't find folders and a file called `_index.md` with the rest of the page getting generated automagically very fun to look at, or simple to understand. In the end, I decided to just roll my own. How hard could it possibly be? I needed an excuse to use Tailwind and I had read about a markdown library in Go that allowed you to override the default markdown renderer for your own components. I was also familiar with Go's templating engine because of [an earlier project](https://github.com/boonsboos/biosboons) and with that knowledge, I got to work.

### A new adventure

Coming from Bootstrap and handmade CSS, Tailwind was a breath of fresh air. Very minty. The kind that hurts your eyes when you breathe out while wearing a mask. The documentation for starting from scratch was not very obvious - apparently you have to add `@tailwind` and include some of its things in your base stylesheet - but I got over that initial hurdle quite quickly: with thanks to StackOverflow and actually reading the docs again. RTFM saving the day once again, who would have thought!

Next up on the list was actually designing my pages. Before that, however, I started adding some basic dark/light mode colors to prevent my eyes from burning while working on this in my spare time, which somehow always ends up being ghosting hour. That aside, the homepage looked empty. Since this is my personal portfolio website, I needed to show off some of my projects. Not that any of them are very useful in a real world setting, but some nerd might look at them and think they are cool. Maybe recruiters will, too.

The image you see on the front page was a pain to get right, with just Tailwind classes. And while I could definitely have written my own CSS, sunken-cost had long set in and I persevered. In the end, it looks quite slick if I may say so myself. Perhaps there is space left to fill, but that is for another day.

While for the front page I did actually write all the HTML myself, for the blog I was going to write just the bare frame in which to place the post. Doesn't need to be fancy (yet), just needs to work.

With Go's templating engine I managed to make a very simple overview page that scrolls infinitely and also loads every post I've ever written. 

```html
<div>
    <h2>Posts</h2>
    <div>
        <!-- loops over all the posts -->
        {{ range .Posts }}
        <!-- loads the card with the supplied post data -->
            {{ template "tmpl/postcard" . }}
        {{ end }}
    </div>
</div>
```
> It's really that simple!

### Markdown

Now, to actually load the posts in. I started by first making a sort of in-memory cache for my posts. Hitting disk everytime you open a page is not very efficient. While I have an SSD on my development machine, my server host might not. This also allows me to easily reload files.

I set up an anonymous function with a goroutine that runs the refresh every so often. This seems to work pretty well.

I'm using a git repository containing the posts separately, so that whenever I make a new post, I only have to run a single command. This solution, while not completely hands-free, is hilariously simple and it also leaves the posts on a different platform for safekeeping and people who aren't down for reading my blabberings on my website.

Using [gomarkdown](https://github.com/gomarkdown/markdown) I was able to easily render my posts to HTML. A bit of styling later and voila! Almost done: still need to add syntax highlighting. I read about a Go library that does this. After blatantly copying the example code into my project I expected it to work.

Right?

...right?

It didnt. It turns out [Chroma](https://github.com/alecthomas/chroma) adds CSS classes to the HTML for highlighting. I thought I would use Tailwind to implement these, but Tailwind removes classes that it does not see used in the HTML source. It was unable to find references to Chroma's classes because the content of the posts is generated at runtime. After finding out about Tailwind's [safelist](https://tailwindcss.com/docs/content-configuration#safelisting-classes) I was able to use my own custom theme - which is not particularly pretty and still needs a lot of work, but it gets the job done.

There was only one thing left to do, and that was to sort the posts by date. Go's [io/fs.WalkDir](https://pkg.go.dev/io/fs#WalkDir) function walks the directory in alphabetical order, which is not particularly what I want. So, time to implement a sorting algorithm.

```go
sort.Slice(PostInfoCache, func(i, j int) bool {
        
    // ...

    if yearI == yearJ {
        if monthI == monthJ {
            return dayI > dayJ
        } else {
            return monthI > monthJ
        }
    } else {
        return yearI > yearJ
    }
})
```
While not the most pretty code I have ever written, this works surprisingly well! Now the cards on the blog page are sorted from newest to oldest. I might add a button to reverse the sorting.  I might also implement pagination and searching...

That's a problem for future me.

---

### Conclusion

At some point, I will take the entirety of the project and package it more neatly so others might use my simple generator. For now, I think I'm going to focus on working on interesting projects so I can write about it here.

If you would like to talk to me, my Discord username is at the very top of the page, or use this code to join me: rtDPMjJQwd