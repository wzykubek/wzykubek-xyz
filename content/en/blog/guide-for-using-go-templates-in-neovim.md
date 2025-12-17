---
title: Guide For Using Go Templates in Neovim
description: This article provides instructions how to setup Neovim for Go HTML templates. I will explain how to enable proper syntax highlighting and how to format templates.
keywords:
  - golang
  - html
  - templates
  - neovim
  - treesitter
  - formatter
  - gohtmlfmt
  - gotmpl
  - conform
tags:
  - neovim
  - go
date: 2025-12-17T04:21:46
modified: 2025-12-17T04:21:46
layout: article
draft: false
---
I spent the last couple of days trying to set up Neovim in a way that will allow me to edit Go HTML templates in a more comfortable manner. I wasted so much time tinkering around and looking for a solution, so I write this article as a future reference for myself, but maybe someone will find it helpful too.

## File detection
If you ever used templates in Golang, then you probably know that there isn't either standard file extension nor mimetype that developers can use. It is up to you how you'll name a file. There are some widely used extensions, like `.tmpl`, `.gotmpl`, `.go.tmpl`, `html.tmpl`, but e.g. [Hugo](https://gohugo.io) - *which I use for this blog* - reads only templates ending with `.html`. If you run `nvim main.html` it is obvious that Neovim will detect the file as HTML - not `gotmpl`.

That being said, you need to force Neovim to detect your `main.html` as `gotmpl`[^1]. This is pretty easy; you can use regex pattern on the path to the file and check if it's placed inside `templates` directory. You can go any further with this to improve detection even more if you use any other kind of templates, but for my personal setup it is more than enough.

```lua
vim.filetype.add({
	pattern = {
		[".*/template.?/.*%.html.*"] = "gotmpl",
		[".*/layout.?/.*%.html.*"] = "gotmpl",
	}
})
```

[^1]: I've chosen this file type because [Treesitter](https://github.com/nvim-treesitter/nvim-treesitter) has a parser for it

## Syntax highlighting
You will need [nvim-treesitter](https://github.com/nvim-treesitter/nvim-treesitter) for this one. Basic Neovim offers highlighting only for HTML, but now, when you have file detected as `gotmpl` there won't be any highlighting at all.

I assume that you have installed the plugin. You need to ensure that you have `html` and `gopls` parsers installed. However, even if I had them installed they was working only partially - only `gopls` parts were highlighted. 

To resolve the issue you need to add custom **Treesitter query**. I won't tell you what exactly it is, 'cause I don't know either, but this knowledge is not necessary at all. All you need to know is that we adding custom injections. We inject one parser into another, and we'll use both `html` and `gotmpl` parsers for our templates.

Create `after/queries/gotmpl` directory structure inside your Neovim config root[^2] and put the content of the following file into `after/queries/gotmpl/injections.scm`.

```q
;; extends

((text) @injection.content
(#set! injection.language "html")
 (#set! injection.combined))
```

[^2]: Neovim stores it's configuration files in `$XDG_CONFIG_HOME/nvim` or `$HOME/.config/nvim` directory

## LSP
The other issue I've encountered was related to LSP, precisely [`vscode-html-languageservice`](https://github.com/microsoft/vscode-html-languageservice), known as just `html` and [`gopls`](https://go.dev/gopls/). Both servers should be enabled for the template buffer. If you use [nvim-lspconfig](https://github.com/neovim/nvim-lspconfig) - *you probably do* - you can notice that in [config for `gopls`](https://github.com/neovim/nvim-lspconfig/blob/a2bd1cf7b0446a7414aaf373cea5e4ca804c9c69/lsp/gopls.lua#L91) the enabled file type is `gotmpl`, but in [config for `html`](https://github.com/neovim/nvim-lspconfig/blob/a2bd1cf7b0446a7414aaf373cea5e4ca804c9c69/lsp/html.lua#L28) there is `templ` - *yet another one*.

We can easily fix it by overwriting config for `html`:

```lua
vim.lsp.config('html', {
    filetypes = { 'html', 'gotmpl' },
})

vim.lsp.enable('html')
```

Now both LSPs should be enabled for our file.

## Formatting 
This problem was hardest to resolve for me and I spent the whole evening trying to resolve it. Solution is not perfect yet, but at the time of writing this article I didn't found anything better than this, and I think I won't find anything in the nearest feature. 

Our `html` LSP can be used to format templates with `:lua vim.lap.buf.format()`. However, it only understand HTML part of the document and all Go specific code won't be indented. We need other tool for this task. And that's the whole point - lack of tools.

Firstly, I tested [djLint](https://github.com/djlint/djlint), which is formatter for Jinja templates, but there is an adnotation that Go templates are supported also. Unfortunately formatting was weird and my code was less readable after using it than before.

Second thing I tested was [this](https://www.npmjs.com/package/prettier-plugin-go-template) Prettier plugin, but it was even worse experience. I think I just don't like Prettier style of formatting. Moreover, you need to download it from NPM and enable it in config for each project.

Finally, I discovered [gotmplfmt](https://github.com/miekg/gotmplfmt). This is really simple formatter that does not have any advanced features and have some small caveats. However, it works as expected. I've noticed some issues, but because I know Go I fixed them and sent a patch to the author. I've even created [AUR package](https://aur.archlinux.org/packages/gotmplfmt), so if you use Arch Linux you can download this tool from there. Keep in mind that this is *really* simple formatter and it does behave a little bit dumb compared to popular tools, but this is the only solution I can recommend now.

### Configuring in Neovim
I use [conform.nvim](https://github.com/stevearc/conform.nvim) but you can also achieve similar result with other plugins, like [formatter.nvim](https://github.com/mhartington/formatter.nvim). I just prefer `conform`, because it does one job and does it right. As a plugin manager I use [lazy.nvim](https://github.com/folke/lazy.nvim), you need to adjust snippet below if you're not *lazy*.

You need to define own formatter because [gotmplfmt](https://github.com/miekg/gotmplfmt) is not supported by `conform`, but after publishing this article I would send a PR to the author to add it. Fortunately, configuration is really simple. Just define a formatter, set the binary path and use it as a default formatter for `gotmpl`.

For other languages I prefer formatters provided by LSPs, so I use `lsp_format = fallback` for unconfigured file types.

```lua
{
	'stevearc/conform.nvim',
	lazy = false,
	opts = {
		formatters = {
			gotmplfmt = {
				command = "gotmplfmt",
			},
		},
		formatters_by_ft = {
			gotmpl = { "gotmplfmt" },
		},
		default_format_opts = {
			lsp_format = "fallback",
		},
	}
}
```

## Final thoughts 
+ I have a working template formatting before GTA VI.
+ Prettier sucks.
+ Drink water.