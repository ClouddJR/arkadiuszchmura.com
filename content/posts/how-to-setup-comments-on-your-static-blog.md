---
title: How to setup comments on your static blog
date: 2023-05-30
summary: With a tool called utterances, it's very easy to add comments to your website with minimal code and no need for a database.
---

## Introduction

In this post, I'll show you how to setup comments on your static blog with a tool called [utterances](https://utteranc.es/). It's a lightweight, free, and open-source comments widget built on GitHub issues.

You can see what this tool looks like by scrolling to the bottom of the page where the comments section is.

The idea is simple. Every post on your blog will have a dedicated issue on your chosen GitHub repository. Then, every comment on your post will essentially become a comment on that issue. As a result, you won't need dedicated storage for these comments. Everything will be stored in your repository.

The only downside is that this way of adding comments requires users to have a GitHub account. If you have a technical blog or a website, that shouldn't be a problem since the majority of your audience is familiar with this tool. However, if that's not the case, you might want to look for something simpler.

My blog is built with [Hugo](https://gohugo.io/) and a theme called [PaperModX](https://github.com/reorx/hugo-PaperModX), and that's what I'll be showing here. However, given how easy the integration is, you shouldn't have any problems with it when using different tools.

## Integration

Integration with utterances is relatively simple. You have to go to [their website](https://utteranc.es/) and follow a couple of steps mentioned in the configuration section.

First, you choose a repository where you want to store your comments (and add the utterances app to it). Next, you select some options, like how you want to map your posts to issue titles or which theme you want to use on the widget. Lastly, you are given a code snippet containing a script tag that you have to add to your blog's template and position it where you want your comments to appear.

In the case of my blog, I had to add `comments: true` to my `config.yaml` file and then create the `layouts/partials/comments.html` file containing the generated script tag:

```html
<script src="https://utteranc.es/client.js"
        repo="ClouddJR/arkadiuszchmura.com"
        issue-term="pathname"
        label="comments"
        theme="github-light"
        crossorigin="anonymous"
        async>
</script>
```

> Notice how the options you selected in the configuration step match the attributes on the script tag.

### Dark mode

That's pretty much it when it comes to the integration. If you've configured everything correctly, you should have a working system for adding comments to your website.

However, you might have noticed that my blog supports a dark mode that you can toggle by clicking on the icon at the top of the page. With the current configuration in the script tag, the theme used by the utterances widget will always be `github-light`, even when using dark mode on the website.

This definitely doesn't provide the best user experience, so I decided to find a way to set the correct initial theme when loading the page and also change the theme every time the user toggles it on the website.

This is the code that I ultimately used (I placed it right below the script tag for the utterances widget):

```html
<script>
    function updateUtterancesTheme(isLight) {
        const iframe = document.querySelector('.utterances-frame')
        if (iframe) {
            const message = {
                type: 'set-theme',
                theme: isLight ? 'github-light' : 'github-dark'
            }
            iframe.contentWindow.postMessage(message, 'https://utteranc.es')
        }
    }
</script>
```

The above is the function (credit goes to [this person](https://github.com/utterance/utterances/issues/549#issuecomment-907606127)) for updating the theme used by the utterances widget. If the passed `isLight` variable is true, we'll use the `github-light` theme. Otherwise, it's going to be `github-dark`.

I wanted to execute this function as soon as the page loads and right before the utterances `iframe` makes a request. Here's one way of doing it:

```javascript
// Set correct initial utterance theme
function handleMessageEvent(event) {
    if (event.origin !== 'https://utteranc.es') return

    const isLight = !document.body.classList.contains('dark')
    updateUtterancesTheme(isLight)

    removeEventListener('message', handleMessageEvent)
}

addEventListener('message', handleMessageEvent)
```

After these changes, the theme is set correctly after the page loads. The only thing left to do is to update the theme after toggling it using the icon at the top of the page.

Luckily, with PaperModX, it's quite easy to do. I just had to add the `updateUtterancesTheme` function to the array of callbacks that will be invoked every time the theme is changed, like this:

```javascript
// Update utterances theme every time the theme is changed by a user
toggleThemeCallbacks['comments'] = updateUtterancesTheme
```

And that's all! If you want to get the whole picture and see all the changes in one place, [here's the commit](https://github.com/ClouddJR/arkadiuszchmura.com/commit/191666a418713c4ae25fac9b2b1f6659429b84a4) on my blog's repository that introduced utterances. Also, feel free to explore the rest of the code.

## Summary

In this post, I showed you how easy it is to add comments to your blog or website without much effort using a really useful tool called utterances. Additionally, I presented a way of updating the theme used by the utterances widget so that you can correctly adjust it to a dark or light theme on your website. I hope you found that post helpful!
