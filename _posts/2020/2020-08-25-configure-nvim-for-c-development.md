---
layout: post
title: "Configure nvim for C/C++ Development"
date: 2020-08-25 07:42 +0200
---

Hello,

This post is using https://ianding.io/2019/07/29/configure-coc-nvim-for-c-c++-development/ as reference.
But I skip some steps to make it simpler.

# Install coc completer
Because Coc.nvim relies on Node.js to be installed, we need to make sure we have Node.js installed. 
To install Coc.nvim, first install yarn package.

```sh 
npm install -g yarn
```

To add Coc.nvim plugin, as I use vim-plugged, I add it using following method.

```sh 
Plug 'neoclide/coc.nvim', {'do': { -> coc#util#install()}}
```
Then, add everything in https://github.com/neoclide/coc.nvim readme in GitHub. 
and add, this following snippet into your vimrc.

```vim
" TextEdit might fail if hidden is not set.
set hidden

" Some servers have issues with backup files, see #649.
set nobackup
set nowritebackup

" Give more space for displaying messages.
set cmdheight=2

" Having longer updatetime (default is 4000 ms = 4 s) leads to noticeable
" delays and poor user experience.
set updatetime=300

" Don't pass messages to |ins-completion-menu|.
set shortmess+=c

" Always show the signcolumn, otherwise it would shift the text each time
" diagnostics appear/become resolved.
if has("patch-8.1.1564")
  " Recently vim can merge signcolumn and number column into one
  set signcolumn=number
else
  set signcolumn=yes
endif

" Use tab for trigger completion with characters ahead and navigate.
" NOTE: Use command ':verbose imap <tab>' to make sure tab is not mapped by
" other plugin before putting this into your config.
inoremap <silent><expr> <TAB>
      \ pumvisible() ? "\<C-n>" :
      \ <SID>check_back_space() ? "\<TAB>" :
      \ coc#refresh()
inoremap <expr><S-TAB> pumvisible() ? "\<C-p>" : "\<C-h>"

function! s:check_back_space() abort
  let col = col('.') - 1
  return !col || getline('.')[col - 1]  =~# '\s'
endfunction

" Use <c-space> to trigger completion.
if has('nvim')
  inoremap <silent><expr> <c-space> coc#refresh()
else
  inoremap <silent><expr> <c-@> coc#refresh()
endif

" Use <cr> to confirm completion, `<C-g>u` means break undo chain at current
" position. Coc only does snippet and additional edit on confirm.
" <cr> could be remapped by other vim plugin, try `:verbose imap <CR>`.
if exists('*complete_info')
  inoremap <expr> <cr> complete_info()["selected"] != "-1" ? "\<C-y>" : "\<C-g>u\<CR>"
else
  inoremap <expr> <cr> pumvisible() ? "\<C-y>" : "\<C-g>u\<CR>"
endif

" Use `[g` and `]g` to navigate diagnostics
" Use `:CocDiagnostics` to get all diagnostics of current buffer in location list.
nmap <silent> [g <Plug>(coc-diagnostic-prev)
nmap <silent> ]g <Plug>(coc-diagnostic-next)

" GoTo code navigation.
nmap <silent> gd <Plug>(coc-definition)
nmap <silent> gy <Plug>(coc-type-definition)
nmap <silent> gi <Plug>(coc-implementation)
nmap <silent> gr <Plug>(coc-references)

" Use K to show documentation in preview window.
nnoremap <silent> K :call <SID>show_documentation()<CR>

function! s:show_documentation()
  if (index(['vim','help'], &filetype) >= 0)
    execute 'h '.expand('<cword>')
  else
    call CocAction('doHover')
  endif
endfunction

" Highlight the symbol and its references when holding the cursor.
autocmd CursorHold * silent call CocActionAsync('highlight')

" Symbol renaming.
nmap <leader>rn <Plug>(coc-rename)

" Formatting selected code.
xmap <leader>f  <Plug>(coc-format-selected)
nmap <leader>f  <Plug>(coc-format-selected)

augroup mygroup
  autocmd!
  " Setup formatexpr specified filetype(s).
  autocmd FileType typescript,json setl formatexpr=CocAction('formatSelected')
  " Update signature help on jump placeholder.
  autocmd User CocJumpPlaceholder call CocActionAsync('showSignatureHelp')
augroup end

" Applying codeAction to the selected region.
" Example: `<leader>aap` for current paragraph
xmap <leader>a  <Plug>(coc-codeaction-selected)
nmap <leader>a  <Plug>(coc-codeaction-selected)

" Remap keys for applying codeAction to the current buffer.
nmap <leader>ac  <Plug>(coc-codeaction)
" Apply AutoFix to problem on the current line.
nmap <leader>qf  <Plug>(coc-fix-current)

" Map function and class text objects
" NOTE: Requires 'textDocument.documentSymbol' support from the language server.
xmap if <Plug>(coc-funcobj-i)
omap if <Plug>(coc-funcobj-i)
xmap af <Plug>(coc-funcobj-a)
omap af <Plug>(coc-funcobj-a)
xmap ic <Plug>(coc-classobj-i)
omap ic <Plug>(coc-classobj-i)
xmap ac <Plug>(coc-classobj-a)
omap ac <Plug>(coc-classobj-a)

" Use CTRL-S for selections ranges.
" Requires 'textDocument/selectionRange' support of LS, ex: coc-tsserver
nmap <silent> <C-s> <Plug>(coc-range-select)
xmap <silent> <C-s> <Plug>(coc-range-select)

" Add `:Format` command to format current buffer.
command! -nargs=0 Format :call CocAction('format')

" Add `:Fold` command to fold current buffer.
command! -nargs=? Fold :call     CocAction('fold', <f-args>)

" Add `:OR` command for organize imports of the current buffer.
command! -nargs=0 OR   :call     CocAction('runCommand', 'editor.action.organizeImport')

" Add (Neo)Vim's native statusline support.
" NOTE: Please see `:h coc-status` for integrations with external plugins that
" provide custom statusline: lightline.vim, vim-airline.
set statusline^=%{coc#status()}%{get(b:,'coc_current_function','')}

" Mappings for CoCList
" Show all diagnostics.
nnoremap <silent><nowait> <space>a  :<C-u>CocList diagnostics<cr>
" Manage extensions.
nnoremap <silent><nowait> <space>e  :<C-u>CocList extensions<cr>
" Show commands.
nnoremap <silent><nowait> <space>c  :<C-u>CocList commands<cr>
" Find symbol of current document.
nnoremap <silent><nowait> <space>o  :<C-u>CocList outline<cr>
" Search workspace symbols.
nnoremap <silent><nowait> <space>s  :<C-u>CocList -I symbols<cr>
" D
nnoremap <silent><nowait> <space>j  :<C-u>CocNext<CR>
" Do default action for previous item.
nnoremap <silent><nowait> <space>k  :<C-u>CocPrev<CR>
" Resume latest coc list.
nnoremap <silent><nowait> <space>p  :<C-u>CocListResume<CR>
```

Up to this step, we already have coc.nvim installed and some autocomplete may work. 
But we will do more, so our C/C++ development will be more fun. 

Actually, we can also use coc-clangd extension to our Coc.nvim, but because my reference 
is using ccls, I am following him :D

# Install and configure ccls.
To install ccls, I build it from its GitHub and following https://github.com/MaskRay/ccls/wiki/Build steps.
There are some dependencies of ccls, so we need to have them first before building ccls. Here are the dependencies:
- CMake 3.8 or higher.
    - A C++ compiler with C++17 support:
    - Clang 5 or higher.
    - GNU GCC 7.2 or higher (optional,string_view require libstdc++7 or higher)
    - MSVC 2017 or higher (included with VS2017 Build Tools)
- On POSIX systems, either GNU Make or Ninja (optional on other systems)
- Clang+LLVM headers and libraries, version >= 7 (0.20181225.8 is the last release that supports clang 6)

For installing clang, I install it from the prebuilt package for Ubuntu 18.04 using following steps:

```sh
wget -c https://releases.llvm.org/9.0.0/clang+llvm-9.0.0-x86_64-linux-gnu-ubuntu-18.04.tar.xz
tar xf clang+llvm-9.0.0-x86_64-linux-gnu-ubuntu-18.04.tar.xz
cmake -H. -BRelease -DCMAKE_BUILD_TYPE=Release -DCMAKE_PREFIX_PATH=$PWD/clang+llvm-9.0.0-x86_64-linux-gnu-ubuntu-18.04
cmake --build Release
```

I am sorry, I forget the exact step after this :p. But I think, up to this step, we can build ccls using following steps:

```sh
git clone --depth=1 --recursive https://github.com/MaskRay/ccls
cd ccls
cmake -H. -BRelease -DCMAKE_BUILD_TYPE=Release -DCMAKE_PREFIX_PATH=/path/to/clang+llvm-xxx
cmake --build Release
```

# GTK development setup
As I want to learn C and make my app using GTK, I need to configure ccls so the autocomplete works very fine,
and its linter knows the GTK library. Out of the box, ccls only knows standard library inside /usr/include.
I do not know why ccls does not know GTK lib which is also installed in /usr/include.

I add a `.ccls` file in my root project, and in the .ccls file, I copy-paste everything 
between `#include <...> search starts here:` and `End of search`
of the output of `g++ -E -x c++ - -v < /dev/null`. And add GTK library location. 
In short, my `.ccls` is like this:

```sh
-I/usr/include/c++/9
-I/usr/include/x86_64-linux-gnu/c++/9
-I/usr/include/c++/9/backward
-I/usr/lib/gcc/x86_64-linux-gnu/9/include
-I/usr/local/include
-I/usr/include/x86_64-linux-gnu
-I/usr/include
-I/usr/include/gtk-3.0
-I/usr/include/glib-2.0
```
Well, that's it what I can share. I am sorry I forget some steps because I did not
take a note when setting this up. Anyway, have a fun development!


Cheers,\
Rahmanu
