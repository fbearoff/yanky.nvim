# 🍃 yanky.nvim

**This plugin still under development, use at your own risk**

![Lua](https://img.shields.io/badge/Made%20with%20Lua-blueviolet.svg?style=for-the-badge&logo=lua)
[![GitHub Workflow Status](https://img.shields.io/github/workflow/status/gbprod/yanky.nvim/Integration?style=for-the-badge)](https://github.com/gbprod/yanky.nvim/actions/workflows/integration.yml)

The aim of `yanky.nvim` is to improve yank and put functionalities for Neovim.

French slogan:

> "Vas-y Yanky, c'est bon !" - Yanky Vincent

Or in English:

> "Yanky-ki-yay, motherf\*cker" - John McYanky

## ✨ Features

- [Yank-ring](#%EF%B8%8F-yank-ring)
- [Yank history picker](#-yank-history-picker)
- [Highlight put and yanked text](#-highlight-put-and-yanked-text)
- [Perserve cursor position on yank](#%EF%B8%8F-preserve-cursor-position-on-yank)
- [Special put](#%EF%B8%8F-special-put)

## ⚡️ Requirements

Requires neovim > `0.7.0`.

## Usage

## 📦 Installation

Install the plugin with your preferred package manager:

### [packer](https://github.com/wbthomason/packer.nvim)

```lua
-- Lua
use({
  "gbprod/yanky.nvim",
  config = function()
    require("yanky").setup({
      -- your configuration comes here
      -- or leave it empty to use the default settings
      -- refer to the configuration section below
    })
  end
})
```

### [vim-plug](https://github.com/junegunn/vim-plug)

```vim
" Vim Script
Plug 'gbprod/yanky.nvim'
lua << EOF
  require("yanky").setup({
    -- your configuration comes here
    -- or leave it empty to use the default settings
    -- refer to the configuration section below
  })
EOF
```

## ⚙️ Configuration

Yanky comes with the following defaults:

```lua
{
  ring = {
    history_length = 10,
    storage = "shada",
    sync_with_numbered_registers = true,
    cancel_event = "update",
  },
  picker = {
    select = {
      action = nil, -- nil to use default put action
    },
    telescope = {
      mappings = nil, -- nil to use default mappings
    },
  },
  system_clipboard = {
    sync_with_ring = true,
  },
  highlight = {
    on_put = true,
    on_yank = true,
    timer = 500,
  },
  preserve_cursor_position = {
    enabled = true,
  },
}
```

### ⌨️ Mappings

This plugin contains no default mappings and will have no effect until you add your own maps to it.
You should at least set those keymaps:

```lua
vim.keymap.set({"n","x"}, "p", "<Plug>(YankyPutAfter)")
vim.keymap.set({"n","x"}, "P", "<Plug>(YankyPutBefore)")
vim.keymap.set({"n","x"}, "gp", "<Plug>(YankyGPutAfter)")
vim.keymap.set({"n","x"}, "gP", "<Plug>(YankyGPutBefore)")
```

Some features requires specific mappings, refer to feature documentation section.

## 🖇️ Yank-ring

Yank-ring allows cycling throught yank history when putting text (like the Emacs
"kill-ring" feature). Yanky automatically maintain a history of yanks that you
can choose between when pasting.

### ⌨️ Mappings

```lua
vim.keymap.set("n", "<c-n>", "<Plug>(YankyCycleForward)")
vim.keymap.set("n", "<c-p>", "<Plug>(YankyCycleBackward)")
```

With these mappings, after performing a paste, you can cycle through the history
by hitting `<c-n>` and `<c-p>`. Any modifications done after pasting will cancel
the possibility to cycle.

Note that the swap operations above will only affect the current paste and the
history will be unchanged.

### ⚙️ Configuration

```lua
require("yanky").setup({
  ring = {
    history_length = 10,
    storage = "shada",
    sync_with_numbered_registers = true,
    cancel_event = "update",
  },
  system_clipboard = {
    sync_with_ring = true,
  },
})
```

#### `ring.history_length`

Default : `10`

Define the number of yanked items that will be saved and used for ring.

#### `ring.storage`

Default : `shada`

Available : `shada` or `memory`

Define the storage mode for ring values.

Using `shada`, this will save pesistantly using Neovim ShaDa feature. This means
that history will be persisted between each session of Neovim.

You can also use this feature to sync the yank history across multiple running instances
of Neovim by updating shada file. If you execute `:wshada` in the first instance
and then `:rshada` in the second instance, the second instance will be synced with
the yank history in the first instance.

Using `memory`, each Neovim instance will have his own history and il will be
lost between sessions.

### `ring.sync_with_numbered_registers`

Default : `true`

History can also be synchronized with numbered registers. Every time the yank
history changes the numbered registers 1 - 9 will be updated to sync with the
first 9 entries in the yank history. See [here](http://vimcasts.org/blog/2013/11/registers-the-good-the-bad-and-the-ugly-parts/)
for an explanation of why we would want do do this.

### `ring.cancel_event`

Default: `update`

Define the event used to cancel ring activation. `update` will cancel ring on
next buffer update, `move` will cancel ring when moving cursor or content
changed.

### `system_clipboard.sync_with_ring`

Default: `true`

Yanky can automatically adds to ring history yanks that occurs outside of Neovim.
This works regardless to your `&clipboard` setting.

This means, if `&clipboard` is set to `unnamed` and/or `unnamedplus`, if you yank
something outside of Neovim, you can put it immediatly using `p` and it will be
added to your yank ring.

If `&clipboard` is empty, if you yank something outside of Neovim, this will be
the first value you'll have when cycling through the ring. Basicly, you can do
`p` and then `<c-p>` to paste yanked text.

### Commands

You can clear yank history using `YankyClearHistory` command.

## 📜 Yank history picker

This allows you to select an entry in your recorded yank history using default
`vim.ui.select` neovim prompt (you can use [stevearc/dressing.nvim](https://github.com/stevearc/dressing.nvim/)
to customize this) or the awesome [telescope.nvim](https://github.com/nvim-telescope/telescope.nvim).

It uses the same history as yank ring, so, if you want to increase history size,
just use [`ring.history_length` option](#ringhistory_length).

### ⚙️ Configuration

To use `vim.ui.select` picker, just call `YankyRingHistory` command.

To use `Telescope` extension, you must first register this extention and then
you can call `Telescope yank_history` :

```lua
require("telescope").load_extension("yank_history")
```

If you want dynamic title with register type in Telescope preview window, you
should set [`dynamic_preview_title`](https://github.com/nvim-telescope/telescope.nvim/blob/master/doc/telescope.txt)
Telescope options to `true`.

Default configuration :

```lua
require("yanky").setup({
  picker = {
    select = {
      action = nil, -- nil to use default put action
    },
    telescope = {
      mappings = nil, -- nil to use default mappings
    },
  },
})
```

#### `picker.select.acion`

Default : `nil`

This define the action that should be done when selecting an item in the
`vim.ui.select` prompt. If you let this option to `nil`, this will use the
default action : put selected item after cursor.

Available actions:

```lua
require("yanky.picker").actions.put("p") -- put after cursor
require("yanky.picker").actions.put("P") -- put before cursor
require("yanky.picker").actions.put("gp") -- put after cursor and leave the cursor after
require("yanky.picker").actions.put("gP") -- put before cursor and leave the cursor after
require("yanky.picker").actions.delete() -- delete entry from yank history
require("yanky.picker").actions.set_register(regname) -- fill register with selected value
```

#### `picker.telescope.mappings`

Default : `nil`

This define the mappings available in Telescope. If you let this option to `nil`,
this will use the default mappings :

```lua
local utils = require("yanky.utils")
local mapping = require("yanky.telescope.mapping")

require("yanky").setup({
  picker = {
    telescope = {
      mappings = {
        default = mapping.put("p"),
        i = {
          ["<c-p>"] = mapping.put("p"),
          ["<c-k>"] = mapping.put("P"),
          ["<c-x>"] = mapping.delete(),
          ["<c-r>"] = mapping.set_register(utils.get_default_register()),
        },
        n = {
          p = mapping.put("p"),
          P = mapping.put("P"),
          d = mapping.delete(),
          r = mapping.set_register(utils.get_default_register())
        },
      }
    }
  }
})
```

Available actions:

```lua
require("yanky.telescope.mapping").put("p") -- put after cursor
require("yanky.telescope.mapping").put("P") -- put before cursor
require("yanky.telescope.mapping").put("gp") -- put after cursor and leave the cursor after
require("yanky.telescope.mapping").put("gP") -- put before cursor and leave the cursor after
require("yanky.telescope.mapping").delete() -- delete entry from yank history
require("yanky.telescope.mapping").set_register(regname) -- fill register {regname} with selected value
```

[_NB: More actions and mappings will come._](https://github.com/gbprod/yanky.nvim/issues/3)

## 💡 Highlight put and yanked text

This will give you a visual feedback on put and yank text
by highlighting this.

### Configuration

```lua
require("yanky").setup({
  highlight = {
    on_put = true,
    on_yank = true,
    timer = 500,
  },
})
```

You can override `YankyPut` highlight to change colors.

#### `highlight.on_put`

Default : `true`

Define if highlight put text feature is enabled.

#### `highlight.on_yank`

Default : `true`

Define if highlight yanked text feature is enabled.

#### `ring.timeout`

Default : `500`

Define the duration of highlight.

## ⤵️ Preserve cursor position on yank

By default in Neovim, when yanking text, cursor moves to the start of the yanked
text. Could be annoying especially when yanking a large text object such as a
paragraph or a large text object.

With this feature, yank will function exactly the same as previously with the one
difference being that the cursor position will not change after performing a yank.

### ⌨️ Mappings

```lua
vim.keymap.set({"n","x"}, "y", "<Plug>(YankyYank)")
```

### ⚙️ Configuration

```lua
require("yanky").setup({
  preserve_cursor_position = {
    enabled = true,
  },
})
```

#### `preserve_cursor_position.enable`

Default : `true`

Define if cursor position should be preserved on yank. This works only if mappings
has been defined.

## 🎨 Colors

| Description                     | Group       | Default        |
| ------------------------------- | ----------- | -------------- |
| Highlight color for put text    | YankyPut    | link to Search |
| Highlight color for yanked text | YankyYanked | link to Search |

## 🤝 Integrations

<details>
<summary><b>gbprod/substitute.nvim</b></summary>

To enable [gbprod/substitute.nvim](https://github.com/gbprod/substitute.nvim)
swap when performing a substitution, you can add this to your setup:

```lua
require("substitute").setup({
  on_substitute = function(event)
    require("yanky").init_ring("p", event.register, event.count, event.vmode:match("[vV]"))
  end,
})
```

</details>

## ↩️ Special put

Yanky comes with special put moves (inspired by
[tpope/vim-unimpaired](https://github.com/tpope/vim-unimpaired/blob/master/doc/unimpaired.txt#L100)):

- Linewise put: this will force put above or below the current line ;
- Shift right/left put: will put above or below the current line and increasing
  or decreasing indent ;
- Filter put: will put above or below the current line and reindenting.

### ⌨️ Mappings

For basic usage (like with [tpope/vim-unimpaired](https://github.com/tpope/vim-unimpaired/blob/master/doc/unimpaired.txt#L100)),
you can use those bindings:

```lua
vim.keymap.set("n", "]p", "<Plug>(YankyPutIndentAfterLinewise)")
vim.keymap.set("n", "[p", "<Plug>(YankyPutIndentBeforeLinewise)")
vim.keymap.set("n", "]P", "<Plug>(YankyPutIndentAfterLinewise)")
vim.keymap.set("n", "[P", "<Plug>(YankyPutIndentBeforeLinewise)")

vim.keymap.set("n", ">p", "<Plug>(YankyPutIndentAfterShiftRight)")
vim.keymap.set("n", "<p", "<Plug>(YankyPutIndentAfterShiftLeft)")
vim.keymap.set("n", ">P", "<Plug>(YankyPutIndentBeforeShiftRight)")
vim.keymap.set("n", "<P", "<Plug>(YankyPutIndentBeforeShiftLeft)")

vim.keymap.set("n", "=p", "<Plug>(YankyPutAfterFilter)")
vim.keymap.set("n", "=P", "<Plug>(YankyPutBeforeFilter)")
```

To go further, Plug mappings are constructed like this: `Yanky(put-type)(modifier)`.

`put-type` can be:

- `PutAfter`: put after your cursor (as [`p`](https://neovim.io/doc/user/change.html#put) key) ;
- `PutBefore`: put before your cursor (as [`P`](https://neovim.io/doc/user/change.html#P) key) ;
- `GPutAfter`: like `PutAfter` but leave the cursor after the new text (as [`gp`](https://neovim.io/doc/user/change.html#gp) key) ;
- `GPutBefore`: like `PutBefore` but leave the cursor after the new text (as [`gP`](https://neovim.io/doc/user/change.html#gP) key) ;
- `PutIndentAfter`: like `PutAfter` but adjust the indent to the current line (as [`]p`](https://neovim.io/doc/user/change.html#]p) key) ;
- `PutIndentBefore`: like `PutBefore` but adjust the indent to the current line (as [`[p`](https://neovim.io/doc/user/change.html#[p) key) ;

`modifier` (optional) can be:

- `Linewise`: put in linewise mode ;
- `ShiftRight`: increase indent ;
- `ShiftLeft`: decrease indent.

## 🎉 Credits

This plugin is mostly a lua version of [svermeulen/vim-yoink](https://github.com/svermeulen/vim-yoink)
awesome plugin.

Other inspiration :

- [bfredl/nvim-miniyank](https://github.com/bfredl/nvim-miniyank)
- [maxbrunsfeld/vim-yankstack](https://github.com/maxbrunsfeld/vim-yankstack)
- [svermeulen/vim-easyclip](https://github.com/svermeulen/vim-easyclip)
- [bkoropoff/yankee.vim](https://github.com/bkoropoff/yankee.vim)
- [svban/YankAssassin.vim](https://github.com/svban/YankAssassin.vim)
- [tpope/vim-unimpaired](https://github.com/tpope/vim-unimpaired)

Thanks to [m00qek lua plugin template](https://github.com/m00qek/plugin-template.nvim).
