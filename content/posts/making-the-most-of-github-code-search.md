---
title: Making the most of GitHub Code Search
date: 2023-09-30
summary: The new search engine offers powerful code searching mechanisms we can leverage for quickly finding the code we need, as well as for exploration and learning.
---

## Introduction

The GitHub Code Search tool was launched as a technical preview in December 2021. Almost 1.5 years later, in May 2023, it was made generally available to all GitHub users.

Fortunately, I was one of the developers who got access to the early versions of the new search engine. Months of exploration and largely positive experience led me to include this tool as an essential part of my daily workflow. 

In this post, I will explain what makes this tool great not merely for searching code, but also for exploration and learning.

## Overview of GitHub Code Search

You can access the new search tool by clicking on the search bar at the top of the GitHub website, by pressing the `/` character, or by going straight to https://github.com/search.

The documentation for code search syntax can be found
[under this link](https://docs.github.com/en/search-github/github-code-search/understanding-github-code-search-syntax), but here's a quick overview of what you can do: 

* Searching for `index` will return files containing the term "index" in their content or path. So it would return both `public/index.php`, even if it doesn't contain the term "index", and `public/sitemap.xml`, if it does.
* Search supports boolean expressions. You can use `AND`, `OR`, and `NOT` operators. For example, the `bitmap OR index` query will return files containing the term "bitmap" or "index", in any order. The `bitmap index` query uses an explicit `AND` operator, so it's equivalent to `bitmap AND index`. Parenthesis can be used to compose more complex expressions, like `bitmap AND (index OR header)`.
* To search for an exact string, including whitespace, you can surround the query with double quotes. If your query contains quotation marks, you can escape them with a backslash. For instance, to search for the `"license": "MIT"` string, we could use the `"\"license\": \"MIT\""` query.
* It's possible to use regular expressions. You can do that by surrounding the expression with slashes. For example, to search for Polish postal codes, we could use the `/\d{2}-\d{3}/` query.
* You can narrow your search using special qualifiers, like `org:github`, `user:ClouddJR`, `language:ruby`, `path:*.txt` (notice the `path:` qualifier supports glob expressions), or `symbol:HelloController`. You can see the full list of available qualifiers with an explanation [here](https://docs.github.com/en/search-github/github-code-search/understanding-github-code-search-syntax#using-qualifiers).

Results for code search are always restricted to 100 results (5 pages). At the time of writing this post, sorting is not supported, although some heuristics are used to prioritize more meaningful results. In most cases, you can expect results from popular repositories to appear first.

There are other limitations worth keeping in mind, like the fact that large repositories might not be indexed at all. The same applies to long lines (over 1024 characters) and files (over 350 KiB). [Here](https://docs.github.com/en/search-github/github-code-search/about-github-code-search#limitations) you can find a list of all current limitations.

### Saved searches

If you find yourself typing the same query over and over again, you might want to use the mechanism for saving searches. 

To do that, type `saved:` in the search bar and click on "Manage saved searches". You can add your query there, which will be visible in the "Saved queries" section when you later access the search bar.

### Advanced search page

This post focuses specifically on GitHub Code Search. If you want to search for non-code content, like issues, you can go to the [advanced search page](https://github.com/search/advanced). This page provides a visual interface for constructing search queries so that you don't have to remember all the available options.

However, if you prefer to type your queries manually, [here's](https://docs.github.com/en/search-github/searching-on-github) a documentation for qualifiers that can be used when searching for non-code content.

Remember that you can use the same search bar for both code and non-code searches, although the syntax might differ.

## Examples

After using the new code search for some time, I found myself using some features more frequently than others. This section will show you the most popular use cases for this tool in my workflow.

### Learning a new API

When learning something new, it makes sense to see some practical examples. They can often be found in the official documentation, but that is not always the case. 

Using the search engine, you can find real-life examples of the technology you're exploring. For example, let's say you're learning SwiftUI and would like to see some examples of how to use the `NavigationStack` in practice. 

You can do that with this simple query: `"NavigationStack {" language:Swift`.

This approach has another benefit. While searching for the code you need, you might find some interesting repositories you were unaware of.

### Searching for options

Recently, I tried to extend the size of my shell's history. I knew the options I had to modify in my `.zshrc` file, but I didn't know what values are used for these options in practice. 

A quick search across the GitHub repositories with the `HISTFILESIZE language:Shell` query showed me what values are common in many dotfiles repositories.

### Inspiration for key bindings

If you want to find out how people configure their tools, like editors, using the search engine can be a great option. 

For instance, to see what keymaps people use for finding files using [Telescope](https://github.com/nvim-telescope/telescope.nvim) in [Neovim](https://neovim.io/), you could search for `builtin.find_files language:Lua`.

### README files

Writing good documentation is hard. Therefore, when doing this ourselves, it might make sense to get some inspiration by reading some well-written README files available on GitHub. 

By using the `path:/(^|\/)README\.md$/` query, we can find some `README.md` files from popular repositories. But obviously, the popularity of a repository doesn't always have to correlate to the quality of its README file.

### Definition of a class

The `symbol:` qualifier is one of the most useful ones. You could use it to find the definition of a specific class (or any other symbol, like a function) when you don't know the repository the class is in. The query could look like this: `symbol:HelloController org:github language:Ruby`.

This way of searching is especially handy when your company uses a microservices architecture and the code is distributed across hundreds (or thousands) of repositories.

You can check out [this link](https://docs.github.com/en/search-github/github-code-search/understanding-github-code-search-syntax#symbol-qualifier) for a list of supported languages for symbol searching.

### Sensitive information

Unfortunately, with the power of the new search tool, it's now almost effortless to find repositories that contain sensitive information (like API keys for Firebase, etc.)

The `"\"current_key\": " path:google-services.json` query reveals a lot of unprotected repositories. 

> Note that even though the `google-services.json` file itself is considered safe to share with other people, it may contain additional information about your resources, like your database URL. If your security rules are not configured properly, an attacker could use that against you.

However, we could use this capability to find repositories in our organization that expose sensitive information and address these vulnerabilities immediately.

## Additional resources

I hope you found this post useful. If you have some questions, leave a comment here or reach out to me on [Twitter](https://twitter.com/ClouddJR/).

If you'd like to learn more about the technology behind the new search tool, here are some additional resources I recommend:

* https://github.blog/2023-02-06-the-technology-behind-githubs-new-code-search/ - a high-level overview of the design and architecture of the new code search.
* https://github.blog/2021-12-15-a-brief-history-of-code-search-at-github/ - a brief history of previous attempts at building a search engine suitable for GitHub's scale and an explanation of why building from scratch was necessary despite existing open-source solutions.