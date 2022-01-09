+++
title = "Editing OpenFOAM cases with (Neo)VIM"
outputs = ["Reveal"]
+++

## No IDE no headache;<br>(Neo)VIM for the lazy Foamer

<div style="height: 10%;"></div>

---

### The stable stuff!

We'll walk you through setting up your NeoVIM to work **_more_ effectively**
with OpenFOAM case files!

- Here is an example [NeoVim config](https://github.com/FoamScience/openfoam-nvim).
- There is also a `foamscience/foam-language-server:latest` Docker image which has all required software.

---

##### Stable features

1. Fast and fault-tolerant syntax highlighting using [TreeSitter grammar](https://github.com/FoamScience/tree-sitter-foam)
    - Native handling of C++ code blocks
2. Hustle-free folding
3. Syntax-aware text objects
4. Smart syntax-aware text subjects
5. Context display

---

#### Notes to remember

- Every feature has a demo, make sure to scroll down to view it
- NeoVIM-specific technology is used here but everything
  is easy to replicate in other editors, or even with (Neo)VIM using other plugins

- You need at least [NeoVim 0.6.0+](https://github.com/neovim/neovim/releases)

---

## 1. Syntax highlighting

<section data-markdown>
<script type="text/template">
- Install  the [nvim-treesitter/nvim-treesitter](https://github.com/nvim-treesitter/nvim-treesitter) plugin
- In NeoVIM, go `:TSInstall foam`
    - It pulls the [TreeSitter grammar](https://github.com/FoamScience/tree-sitter-foam)
      for OpenFOAM and compiles it; That's all you need to do
</script>
</section>

<section data-markdown>
<script type="text/template">

How is the file type detected?

- The plugin is [doing its best](https://github.com/nvim-treesitter/nvim-treesitter/blob/master/after/ftdetect/foam.vim) for common OpenFOAM file names
- But It's recommended to add (in your `after/ftdetect/foam.vim` for example):

```vim
autocmd BufNewFile,BufRead,BufEnter * call s:foamFile(expand("%"))
```
</script>
</section>

{{< section >}}
<img style="width:90%; margin: 8% 0 0 0; padding:0;" data-src="images/syntax_highlighting_01.png">
{{< /section >}}

---

### Highlighting C++ code blocks

<section data-markdown>
<script type="text/template">
- `:TSInstall cpp`
    - The OpenFOAM grammar will send C++ bits to the C++ parser!
    - If you `:TSInstall regex`, it will also highlight regular expressions!
</script>
</section>

{{< section >}}
<img style="width:90%; margin: 8% 0 0 0; padding:0;" data-src="images/syntax_highlighting_02.png">
{{< /section >}}

---

## Expression-based folding

<section data-markdown>
<script type="text/template">
- `:set foldmethod=expr`
- `:set foldexpr=nvim_treesitter#foldexpr()`
- That's it! Put these in auto-cmds if you want their behavior to
  persistent (you might want to set `foldminlines`)
</script>
</section>

{{< section >}}
<img style="width:90%; margin: 8% 0 0 0; padding:0;" data-src="images/folding.png">
{{< /section >}}

---

## Syntax-aware text objects

<section data-markdown>
<script type="text/template">
- Install the [nvim-treesitter-textobjects](https://github.com/nvim-treesitter/nvim-treesitter-textobjects) plugin
- Its default queries provide captures for:
  - Inner and Outer Key-value pairs (called `@function.*`)
  - Inner and Outer dictionaries (called `@class.*`)
  - Single comment (`comment.inner`)
  - What's between two comments (`comment.outer`)
</script>
</section>

<section data-markdown>
<script type="text/template">
Configs for **selecting**, **moving** and **swapping** text objects:
```lua
local ts_config = require('nvim-treesitter.configs')
ts_config.setup {
  textobjects = {
    select = {
      	enable = true,
      	lookahead = true,
          -- iF and aF families for selecting text objects
      	keymaps = {
      		-- v keymaps are for key-values
      	  	["av"] = "@function.outer",
      	  	["iv"] = "@function.inner",
  		    -- c keymaps are for dictionaries
      	  	["ac"] = "@class.outer",
      	  	["ic"] = "@class.inner",
  		    -- k keymaps are for comments
      	  	["ak"] = "@comment.outer",
      	  	["ik"] = "@comment.inner",
      	},
    },
    move = {
      	enable = true,
      	set_jumps = true,
  	    -- Granular control over motions on key-values and dictionaries
  	    -- if you want it
      	goto_next_start = {
      	  	["]m"] = "@function.outer",
      	  	["]]"] = "@class.outer",
      	},
      	goto_next_end = {
      	  	["]M"] = "@function.outer",
      	  	["]["] = "@class.outer",
      	},
      	goto_previous_start = {
      	  	["[m"] = "@function.outer",
      	  	["[["] = "@class.outer",
      	},
      	goto_previous_end = {
      	  	["[M"] = "@function.outer",
      	  	["[]"] = "@class.outer",
      	},
    },
  	swap = {
  	  	enable = true,
  	  	-- Swap parameters; ie. if a key-value has multiple values
        -- and you want to swap them
  	  	swap_next = {
  	  	  	[",a"] = "@parameter.inner",
  	  	},
  	  	swap_previous = {
  	  	  	[",A"] = "@parameter.inner",
  	  	},
  	},
  },
}
```
</script>
</section>

{{< section >}}
<video style="display:block;width:85%; margin:100px auto;" data-autoplay src="videos/textobjects.webm"></video>
{{< /section >}}

---

## Smart Text subjects

<section data-markdown>
<script type="text/template">
- Install the [nvim-treesitter-textsubjects](https://github.com/RRethy/nvim-treesitter-textsubjects) plugin
- Allows for incremental and smart selection of:
  - Parts of keyword values
  - Key-value pairs and list elements
  - Dictionary body 
</script>
</section>

<section data-markdown>
<script type="text/template">
Sample configuration for smart text subjects
```lua
local ts_config = require('nvim-treesitter.configs')
ts_config.setup {
  textsubjects = {
      enable = true,
      keymaps = {
  		-- Press v. inside a key-value or a comment (then . or ; repeatedly)
          ['.'] = 'textsubjects-smart',
  		-- Press v; to select surrounding dictionary
          [';'] = 'textsubjects-container-outer',
      }
  },
}
```
</script>
</section>

{{< section >}}
<video style="display:block; width:85%; margin:100px auto;" data-autoplay src="videos/textsubjects.webm"></video>
{{< /section >}}

---

## Context display

<section data-markdown>
<script type="text/template">
- Install the [nvim-treesitter-context](https://github.com/romgrk/nvim-treesitter-context) plugin
- Shows you the current dictionary or key-value pair 
  - Only if it's long enough to span over multiple screen pages
</script>
</section>

<section data-markdown>
<script type="text/template">
Set it up in the following way:
```lua
require("treesitter-context").setup({
    throttle = true,
    patterns = {
        default = { 'function', 'class', 'method' },
        foam = { '^dict$', '^key_value$' }
    },
    -- Make sure foam is treated with exact patterns
    exact_patterns = { foam = true, }
})
```
</script>
</section>

{{< section >}}
<video style="display:block; width:85%; margin:100px auto; padding:0;" data-autoplay src="videos/context-display.webm"></video>
{{< /section >}}

---

### The UNstable stuff!

> LSP features are still under development; but kind of useful already.

- Expect frequent changes
- Here, we're using NeoVim's native LSP client
- The following works also with VS-Code through [this extension](https://github.com/FoamScience/foam-language-client)

---

#### Get the language server working!

<section data-markdown>
<script type="text/template">
1. Install [neovim/nvim-lspconfig](https://github.com/neovim/nvim-lspconfig)
    - This is an interface to the native LSP client
    - Which provides sane configuration for many language servers
2. Install [williamboman/nvim-lsp-installer](https://github.com/williamboman/nvim-lsp-installer)
    - Seamless server installation (`:LspInstall foam`)
</script>
</section>

<section data-markdown>
<script type="text/template">
- Dig around [init.vim#L145](https://github.com/FoamScience/openfoam-nvim/blob/e9a6a3a6c303a066eec541e4a1fac0260680ff87/init.vim#L145) for an example configuration with auto-completion and snippets support
- `:LspInfo` should say:
    - the client is attached with the correct file types
    - the correct case directory (as `root dir`) is selected
</script>
</section>

{{< section >}}
<img style="width:90%; margin: 8% 0 0 0; padding:0;" data-src="images/lsp-info.png">
{{< /section >}}

---

#### Before showing you the features

- Please [file bug reports](https://github.com/FoamScience/foam-language-server/issues) if any unwanted behavior is observed.
- Or even better, solve the issues, and PR!
- You're also welcome to [discuss](https://github.com/FoamScience/foam-language-server/discussions)
    any suggestions or feature requests you/others might have.
- You probably have all the following commands mapped to some keys, find them with `:map`

---

### Auto-Completion

<section data-markdown>
<script type="text/template">
- Auto-Completing macro-expansion variables on '$'
- Suggesting preprocessor-like directives on '#'
- Suggesting common key-value pairs [Few basic ones for now]
- Built-in snippets [Few basic ones for now]
</script>
</section>

{{< section >}}
<video style="display:block; width:85%; margin:100px auto; padding:0;" data-autoplay src="videos/auto-complete.webm"></video>
{{< /section >}}

---

### Jump to definition

<section data-markdown>
<script type="text/template">
- `:lua vim.lsp.buf.definition()` when the cursor is on a macro variable takes you to the __value__ of
    its definition (the key-value pair where it's defined).
- If you have it set up, "peek definition" will also work
</script>
</section>

{{< section >}}
<video style="display:block; width:85%; margin:100px auto; padding:0;" data-autoplay src="videos/jump-definition.webm"></video>
{{< /section >}}

{{< section >}}
<img style="width:70%; margin: 8% 0 0 0; padding:0;" data-src="images/peek_definition.png">
{{< /section >}}

---

### Hover documentation

<section data-markdown>
<script type="text/template">
- `:lua vim.lsp.buf.hover()` shows local documentation for the keyword under the cursor.
    - [A limited number of keywords is supported currently]
</script>
</section>

{{< section >}}
<img style="width:90%; margin: 8% 0 0 0; padding:0;" data-src="images/hover.png">
{{< /section >}}

---

### Signature help

<section data-markdown>
<script type="text/template">
- `:lua vim.lsp.buf.signature_help()` shows local signature help for common OpenFOAM
    keywords
    - [A limited number of keywords is supported currently]
</script>
</section>

{{< section >}}
<img style="width:90%; margin: 8% 0 0 0; padding:0;" data-src="images/signature_help.png">
{{< /section >}}

---

### Document symbols

<section data-markdown>
<script type="text/template">
- Having trouble navigating a huge dictionary?
    - Try `:lua vim.lsp.buf.document_symbol()`
- All key-value pairs and dictionaries in a file are indexed
    - Including the ones which are **inside lists**
    - Symbols have meaningful types (Key, Struct, Number ... etc) for easy filtering with fzf.
</script>
</section>

<section data-markdown>
<script type="text/template">
- If you have multiple buffers open
    - Try `:lua vim.lsp.buf.workspace_symbol()` for workspace-wide view of symbols
    - Always takes you to the **value** of the selected symbol
</script>
</section>

{{< section >}}
<video style="display:block; width:85%; margin:100px auto; padding:0;" data-autoplay src="videos/symbols.webm"></video>
{{< /section >}}

---

### Diagnostics

<section data-markdown>
<script type="text/template">
- This is the **only** feature which requires **OpenFOAM to be sourced**
  (solver from `system/controlDict` must be found on `PATH`)
- Diagnostics are **workspace-wide**
- `:lua vim.diagnostic.open_float()` on erronous lines shows the full error message
- `:lua vim.diagnostic.setqflist()` opens the quick-fix list
</script>
</section>

{{< section >}}
<video style="display:block; width:85%; margin:100px auto; padding:0;" data-autoplay src="videos/diagnostics.webm"></video>
{{< /section >}}

---

## Next steps

---

- Better parsing performance
    - Eventually, switch to native node bindings (using WASM bindings for now).
    - Goal: No waiting time for full-case workspace symbols.
- Better server performance
    - Async-do everything
