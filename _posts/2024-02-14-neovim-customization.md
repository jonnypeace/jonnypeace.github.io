---
title: "Setting up a Custom Neovim config on Arch Linux"
date: 2024-02-14 11:00
categories: [Arch Linux, neovim, config, python, language support]
tags: [Arch Linux, neovim, config, python, language support]
---

## vim-plug

First we need to install (place the plug.vim into the autoload directory) vim-plug.

```bash
curl -fLo ~/.local/share/nvim/site/autoload/plug.vim --create-dirs https://raw.githubusercontent.com/junegunn/vim-plug/master/plug.vim
```

## Install npm and diagnostic language servers

```bash
sudo pacman -S npm
sudo npm install -g pyright yaml-language-server vscode-json-languageserver bash-language-server typescript typescript-language-server vscode-html-languageserver-bin vscode-css-languageserver-bin
```

## Neovim commands

Ok, we can update, and install specific packages, but using our config with TSInstall by itself you will install support for 200+ languages at the time of writing. I'll put the commands here for explicitly installing python.

```
:PlugInstall # installs plugins
:TSInstall python # installs python
:TSUpdate # updates all installed language parsers to their latest versions.
```

## Neovim config

Located $HOME/.config/nvim/init.vim

```vim
call plug#begin('~/.config/nvim/plugged')
" Your Plug commands will go here, for example:
Plug 'nvim-treesitter/nvim-treesitter', {'do': ':TSUpdate'}
Plug 'hrsh7th/nvim-cmp' " The completion engine
Plug 'hrsh7th/cmp-buffer' " Buffer completions
Plug 'hrsh7th/cmp-path' " Path completions
Plug 'hrsh7th/cmp-nvim-lsp' " LSP completions
Plug 'hrsh7th/cmp-nvim-lua' " Neovim Lua API completions
Plug 'neovim/nvim-lspconfig' " Common configurations for the Neovim LSP client
Plug 'hrsh7th/cmp-vsnip' " Snippet completions
Plug 'hrsh7th/vim-vsnip' " Snippet engine
call plug#end()

" install programming language packs
lua << EOF
require'nvim-treesitter.configs'.setup {
  highlight = {
    enable = true,              -- false will disable the whole extension
  },
  ensure_installed = "all"     -- one of "all", "maintained" (parsers with maintainers), or a list of languages
}
EOF

" Install for predicting typing
lua << EOF
local cmp = require'cmp'

cmp.setup({
  snippet = {
    expand = function(args)
      vim.fn["vsnip#anonymous"](args.body) -- For `vsnip` users.
    end,
  },
  mapping = cmp.mapping.preset.insert({
    ['<C-b>'] = cmp.mapping.scroll_docs(-4),
    ['<C-f>'] = cmp.mapping.scroll_docs(4),
    ['<C-Space>'] = cmp.mapping.complete(),
    ['<C-e>'] = cmp.mapping.abort(),
    ['<CR>'] = cmp.mapping.confirm({ select = true }), -- Accept currently selected item. Set `select` to `false` to only confirm explicitly selected items.
  }),
  sources = cmp.config.sources({
    { name = 'nvim_lsp' },
    { name = 'vsnip' }, -- For vsnip users.
    { name = 'buffer' },
    { name = 'path' }
  })
})
EOF

" Powerful Diagnostic Functionality
lua << EOF
-- PYTHON
require'lspconfig'.pyright.setup{}

-- YAML
require'lspconfig'.yamlls.setup{}

-- JSON
require'lspconfig'.jsonls.setup{
  commands = {
    Format = {
      function()
        vim.lsp.buf.range_formatting({}, {0,0}, {vim.fn.line("$"),0})
      end
    }
  }
}

-- Bash
require'lspconfig'.bashls.setup{}

-- JavaScript and TypeScript
require'lspconfig'.tsserver.setup{}

-- HTML
require'lspconfig'.html.setup{}

-- CSS
require'lspconfig'.cssls.setup{}
EOF

" toggle diagnostics
lua << EOF
function ToggleDiagnostics()
    local current_value = vim.diagnostic.config().underline
    vim.diagnostic.config({underline = not current_value, virtual_text = not current_value, signs = not current_value, update_in_insert = not current_value})
end
EOF

" The Leader key defaults to \. You can change it by uncommenting and setting the line below:
" let mapleader = " "
nnoremap <leader>d :lua ToggleDiagnostics()<CR>

" Set tab to 4 spaces
set tabstop=4
set shiftwidth=4
set expandtab

" Always show line numbers
set number

" Toggle relative line numbers with F7
nnoremap <F7> :if &relativenumber == 0 \| set relativenumber \| else \| set norelativenumber \| endif<CR>
```
