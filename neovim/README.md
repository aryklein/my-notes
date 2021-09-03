# Neovim setup

This document describes how I customized my Neovim setup.

## Plugin Manager

To manage my plugins I use [vim-plug](https://github.com/junegunn/vim-plug). To install it, just follow this
[step](https://github.com/junegunn/vim-plug#neovim).

To upgrade vim-plug itself, run: `:PlugUpgrade`.

### How to use it

Define in `~/.config/nvim/init.vim` the list of plugins to use:

```viml
" List of plugins starts here.
call plug#begin()

Plug 'gruvbox-community/gruvbox'
Plug 'hashivim/vim-terraform'
Plug 'vim-airline/vim-airline'
Plug 'vim-airline/vim-airline-themes'

" List ends here.
call plug#end()
```

After adding the above to the top of your Neovim configuration file, reload it (`:source ~/.config/nvim/init.vim`) or
restart Neovim. Now run `:PlugInstall` to install the plugins.

To uninstall a plugin, just remove it or comment out from the configuration file, reload Neovim and execute
`:PlugClean`.

To update all plugins run `:PlugUpdate`.

## Cool plugins that I use

You can check my Neovim [config](https://github.com/aryklein/dotfiles/blob/master/.config/nvim/init.vim) file to get
a complete list of plugins that I'm using. The following plugins are my recommendation:

- `gruvbox-community/gruvbox`: theme with pastel 'retro groove' colors.
- `hashivim/vim-terraform`: adds a `:Terraform` command that runs `terraform`, with tab completion of subcommands. It
also sets up *.hcl*, *.tf*, *.tfvars*, *.terraform.rc*  and *terraform.rc* files to be highlighted as HCL and *.tfstate* as JSON.
- `pearofducks/ansible-vim`: syntax plugin for Ansible.
- `preservim/nerdtree`: a file system explorer.
- `ryanoasis/vim-devicons`: add icons to the plugins.
- `vim-airline/vim-airline`: vim airline bar.
- `vim-airline/vim-airline-themes`: themes for airline bar.
- `airblade/vim-gitgutter`: shows a git diff in the sign column. It shows which lines have been added, modified, or
removed
- `tpope/vim-fugitive`: plugin for Git.
- `yggdroot/indentline`: used for displaying thin vertical lines at each indentation level for code indented with
spaces
- `preservim/nerdcommenter`: add comment functions.
- `nvim-lua/plenary.nvim`, `nvim-telescope/telescope.nvim` and `nvim-telescope/telescope-fzy-native.nvim`: for
Telescope.

### Telescope

Is a highly extendable fuzzy finder over lists. Site [here](https://github.com/nvim-telescope/telescope.nvim)

Install the following plugins:

```viml
Plug 'nvim-lua/plenary.nvim'
Plug 'nvim-telescope/telescope.nvim'
Plug 'nvim-telescope/telescope-fzy-native.nvim'
```

Add the following maps:

```viml
nnoremap <leader>ff <cmd>Telescope find_files<cr>
nnoremap <leader>fg <cmd>Telescope live_grep<cr>
nnoremap <leader>fb <cmd>Telescope buffers<cr>
nnoremap <leader>fh <cmd>Telescope help_tags<cr>
```

**Install the following packages**

- bat
- fd
- ripgrep


**Customizing Telescope**

Embed the following code snippet in a `.vim` file (for example in `~/.config/nvim/after/plugin/telescope.nvim.vim`),
wrap it in `lua << EOF code-snippet EOF`

```lua
lua << EOF
require('telescope').setup{
  defaults = {
    file_ignore_patterns = { '.git/*', 'env/*', '.env/*' },
    file_sorter = require'telescope.sorters'.get_fzy_sorter,
    generic_sorter = require'telescope.sorters'.get_fzy_sorter,
    set_env = { ['COLORTERM'] = 'truecolor' }, -- default = nil,
    file_previewer = require'telescope.previewers'.vim_buffer_cat.new,
    grep_previewer = require'telescope.previewers'.vim_buffer_vimgrep.new,
    qflist_previewer = require'telescope.previewers'.vim_buffer_qflist.new,
    file_ignore_patterns = {".env/*", "%.jpeg", "%.png", "%.jpg", "%.mkv"},
    color_devicons= true,
  },
  extensions = {
    fzy_native = {
      override_generic_sorter = false,
      override_file_sorter = true,
    },
  }
}
-- Load Telescope extensions
require('telescope').load_extension('fzy_native')
EOF
```

