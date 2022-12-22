<div align="center">

# `legendary.nvim`

[Features](#features) | [Prerequisites](#prerequisites) | [Installation](#installation) | [Quickstart](#quickstart) | [Configuration](#configuration)

[![matrix](https://matrix.to/img/matrix-badge.svg)](https://matrix.to/#/%23legendary.nvim:matrix.org)

</div>

Define your keymaps, commands, and autocommands as simple Lua tables, building a legend at the same time (like VS Code's Command Palette).

![demo gif](https://user-images.githubusercontent.com/8648891/200827633-7009f5f3-e126-491c-88bd-73a0287978c4.gif) \
<sup>Theme used in recording is [onedarkpro.nvim](https://github.com/olimorris/onedarkpro.nvim). The finder UI is handled by [telescope.nvim](https://github.com/nvim-telescope/telescope.nvim) via [dressing.nvim](https://github.com/stevearc/dressing.nvim). See [Prerequisites](#prerequisites) for details.</sup>

<details>
<summary>Documentation Table of Contents (click to expand)</summary>

- [Features](#features)
- [Prerequisites](#prerequisites)
- [Installation](#installation)
- [Quickstart](#quickstart)
- [Configuration](#configuration)
- [Keymap Development Utilities](./doc/MAPPING_DEVELOPMENT.md)
- [`which-key.nvim` Integration](./doc/WHICH_KEY.md)
- [Lua API](./doc/API.md)
- [Table Structures](./doc/table_structures/README.md)
  - [Keymaps](./doc/table_structures/KEYMAPS.md)
  - [Commands](./doc/table_structures/COMMANDS.md)
  - [Functions](./doc/table_structures/FUNCTIONS.md)
  - [`augroup`/`autocmd`s](./doc/table_structures/AUTOCMDS.md)

</details>

## Features

- Define your keymaps, commands, `augroup`/`autocmd`s, and even arbitrary Lua functions to run on the fly, as simple Lua tables, then bind them with `legendary.nvim`
- Integration with [which-key.nvim](https://github.com/folke/which-key.nvim), use your existing `which-key.nvim` tables with `legendary.nvim`
- Execute normal, insert, and visual mode keymaps, commands, autocommands, and Lua functions when you select them
- Show your most recently executed items at the top when triggered via `legendary.nvim` (can be disabled via config)
- Uses `vim.ui.select()` so it can be hooked up to a fuzzy finder using something like [dressing.nvim](https://github.com/stevearc/dressing.nvim) for a VS Code command palette like interface
- Buffer-local keymaps, commands, functions and autocmds only appear in the finder for the current buffer
- Help execute commands that take arguments by prefilling the command line instead of executing immediately
- Search built-in keymaps and commands along with your user-defined keymaps and commands (may be disabled in config). Notice some missing? Comment on [this discussion](https://github.com/mrjones2014/legendary.nvim/discussions/89) or submit a PR!
- A `legendary.toolbox` module to help create lazily-evaluated keymaps and commands, and item filter. Have an idea for a new helper? Comment on [this discussion](https://github.com/mrjones2014/legendary.nvim/discussions/90) or submit a PR!
- Sort by [frecency](https://en.wikipedia.org/wiki/Frecency), a combined measure of how frequently and how recently you've used an item from the picker
- A parser to convert Vimscript keymap commands (e.g. `vnoremap <silent> <leader>f :SomeCommand<CR>`) to `legendary.nvim` keymap tables (see [Converting Keymaps From Vimscript](./doc/API.md#converting-keymaps-from-vimscript))
- Anonymous mappings; show mappings/commands in the finder without having `legendary.nvim` handle creating them

## Prerequisites

- (Optional) A `vim.ui.select()` handler; this provides the UI for the finder.
  - I recommend [telescope.nvim](https://github.com/nvim-telescope/telescope.nvim) paired with [dressing.nvim](https://github.com/stevearc/dressing.nvim).

## Installation

This project uses git tags to adhere to [Semantic Versioning](https://semver.org/). To check the latest
version, see the [git tag list](https://github.com/mrjones2014/legendary.nvim/tags).

With `packer.nvim`:

```lua
-- to use a version
use({
  'mrjones2014/legendary.nvim',
  tag = 'v2.1.0',
  -- sqlite is only needed if you want to use frecency sorting
  -- requires = 'kkharji/sqlite.lua'
})
-- or, to get rolling updates
use({
  'mrjones2014/legendary.nvim',
  -- sqlite is only needed if you want to use frecency sorting
  -- requires = 'kkharji/sqlite.lua'
})
```

With `vim-plug`:

```VimL
" if you want to use frecency sorting, sqlite is also needed
Plug "kkharji/sqlite.lua"

" to use a version
Plug "mrjones2014/legendary.nvim", { 'tag': 'v2.1.0' }
" or, to get rolling updates
Plug "mrjones2014/legendary.nvim"
```

## Quickstart

Register keymaps through setup:

```lua
require('legendary').setup({
  keymaps = {
    -- map keys to a command
    { '<leader>ff', ':Telescope find_files', description = 'Find files' },
    -- map keys to a function
    {
      '<leader>h',
      function()
        print('hello world!')
      end,
      description = 'Say hello',
    },
    -- keymaps have opts.silent = true by default, but you can override it
    { '<leader>s', ':SomeCommand<CR>', description = 'Non-silent keymap', opts = { silent = false } },
    -- create keymaps with different implementations per-mode
    {
      '<leader>c',
      { n = ':LinewiseCommentToggle<CR>', x = ":'<,'>BlockwiseCommentToggle<CR>" },
      description = 'Toggle comment',
    },
    -- create item groups to create sub-menus in the finder
    -- note that only keymaps, commands, and functions
    -- can be added to item groups
    {
      -- groups with same itemgroup will be merged
      itemgroup = 'short ID',
      description = 'A submenu of items...',
      icon = '',
      keymaps = {
        -- more keymaps here
      },
    },
  },
  commands = {
    -- easily create user commands
    {
      ':SayHello',
      function()
        print('hello world!')
      end,
      description = 'Say hello as a command',
    },
    {
      -- groups with same itemgroup will be merged
      itemgroup = 'short ID',
      -- don't need to copy the other group data because
      -- it will be merged with the one from the keymaps table
      commands = {
        -- more commands here
      },
    },
  },
  funcs = {
    -- Make arbitrary Lua functions that can be executed via the item finder
    {
      function()
        doSomeStuff()
      end,
      description = 'Do some stuff with a Lua function!',
    },
    {
      -- groups with same itemgroup will be merged
      itemgroup = 'short ID',
      -- don't need to copy the other group data because
      -- it will be merged with the one from the keymaps table
      funcs = {
        -- more funcs here
      },
    },
  },
  autocmds = {
    -- Create autocmds and augroups
    { 'BufWritePre', vim.lsp.buf.format, description = 'Format on save' },
    {
      name = 'MyAugroup',
      clear = true,
      -- autocmds here
    },
  },
})
```

For more mapping features and more complicated setups see [Table Structures](./doc/table_structures/README.md).

To trigger the finder for your configured keymaps, commands, `augroup`/`autocmd`s, and Lua functions:

Commands:

```VimL
" search keymaps, commands, and autocmds
:Legendary

" search keymaps
:Legendary keymaps

" search commands
:Legendary commands

" search functions
:Legendary functions

" search autocmds
:Legendary autocmds
```

Lua API:

The `require('legend').find()` function takes an `opts` table with the following fields (all optional):

```lua
{
  -- pass a list of filter functions or a single filter function with
  -- the signature `function(item): boolean`
  -- several filter functions are provided for convenience
  -- see ./doc/FILTERS.md for a list
  filters = {},
  -- pass a function with the signature `function(item, mode): string[]`
  -- returning a list of strings where each string is one column
  -- use this to override the configured formatter for just one call
  formatter = nil,
  -- pass a string, or a function that returns a string
  -- to customize the select prompt for the current call
  select_prompt = nil,
}
```

See [USAGE_EXAMPLES.md](./doc/USAGE_EXAMPLES.md) for some advanced usage examples.

## Configuration

Default configuration is shown below. For a detailed explanation of the str
