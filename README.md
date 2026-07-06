# init.lua

```
local vim = vim
local Plug = vim.fn['plug#']

vim.g.mapleader = ' '
vim.g.maplocalleader = ' '

vim.opt.wrap = false
vim.opt.ignorecase = true
vim.opt.smartcase = true
vim.opt.hidden = true
vim.opt.expandtab = true
vim.opt.tabstop = 4
vim.opt.shiftwidth = 4
vim.opt.termguicolors = true
vim.cmd.colorscheme('catppuccin')

--- to turn off background for colour scheme ---
local transparent_groups = {
  'Normal',
  'NormalNC',
  'SignColumn',
  'LineNr',
  'CursorLine',
  'CursorLineNr',
  'EndOfBuffer',
  'NormalFloat',
  'FloatBorder',
  'Pmenu',
}

for _, group in ipairs(transparent_groups) do
  vim.api.nvim_set_hl(0, group, { bg = 'NONE' })
end

--- to turn off background for colour scheme ---
-- vim.api.nvim_set_hl(0, 'SnippetTabstop', { bg = 'NONE', underline = true })
-- vim.api.nvim_set_hl(0, 'SnippetTabstopActive', { bg = 'NONE', underline = true })


vim.call('plug#begin')

Plug('ibhagwan/fzf-lua')
Plug('neovim/nvim-lspconfig')
Plug('nvim-treesitter/nvim-treesitter')
Plug('saghen/blink.cmp', { ['tag'] = 'v1.*' })
Plug('rafamadriz/friendly-snippets')

vim.call('plug#end')

--- fzf-lua keybinds ---
local fzf = require('fzf-lua')

vim.keymap.set('n', 'ff', fzf.files, { desc = 'Find files' })
vim.keymap.set('n', 'fg', fzf.live_grep, { desc = 'Live grep' })
vim.keymap.set('n', 'ft', fzf.lsp_document_symbols, { desc = 'Find symbols' })
vim.keymap.set('n', 'q:', '<Nop>')

--- completion configs ---
local blink_ok, blink = pcall(require, 'blink.cmp')
if blink_ok then
  blink.setup({
    keymap = { preset = 'default' },
    completion = {
      documentation = { auto_show = false },
    },
    sources = {
      default = { 'lsp', 'path', 'snippets', 'buffer' },
    },
    snippets = {
      preset = 'default',
    },
    fuzzy = {
      implementation = 'prefer_rust_with_warning',
    },
  })
end

--- treesitter configs ---
require('nvim-treesitter').setup()

vim.api.nvim_create_autocmd('FileType', {
  pattern = { 'erlang' },
  callback = function()
    pcall(vim.treesitter.start)
  end,
})

--- lsp configs ---
vim.diagnostic.config({
  virtual_text = {
    source = 'if_many',
    spacing = 2,
  },
  signs = false,
  underline = true,
  update_in_insert = false,
  severity_sort = true,
})

vim.api.nvim_create_user_command('LspInfo', function()
  local clients = vim.lsp.get_clients({ bufnr = 0 })
  if #clients == 0 then
    print('No LSP clients attached to current buffer')
    return
  end

  for _, client in ipairs(clients) do
    local root = client.config.root_dir or client.root_dir or 'unknown root'
    print(string.format('%s: %s', client.name, root))
  end
end, {})

vim.lsp.config('elp', {
  cmd = { vim.fn.exepath('elp'), 'server' },
  handlers = {
    ['window/showMessage'] = function(_, result, ctx, config)
      if result.type <= vim.lsp.protocol.MessageType.Warning then
        vim.lsp.handlers['window/showMessage'](_, result, ctx, config)
      end
    end,
  },
})
vim.lsp.enable('elp')

vim.api.nvim_create_autocmd('LspAttach', {
  callback = function(event)
    local opts = { buffer = event.buf }

    vim.keymap.set('n', 'gd', vim.lsp.buf.definition, opts)
    vim.keymap.set('n', 'gr', vim.lsp.buf.references, opts)
    vim.keymap.set('n', 'K', vim.lsp.buf.hover, opts)
    vim.keymap.set('n', 'gl', vim.diagnostic.open_float, opts)
    vim.keymap.set('n', '<leader>rn', vim.lsp.buf.rename, opts)
    vim.keymap.set('n', '<leader>ca', vim.lsp.buf.code_action, opts)
  end,
})
```
