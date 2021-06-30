---
title: Vim configuration
slug: vim-configuration
date: 2021-03-13T17:32:19.000Z
date_updated: 2021-03-13T17:32:19.000Z
tags:
  - vim
---

I'm going through some vim configuration to make it feel simpler and easier to use.

## Nerdtree configuration

### Toggle nerdtree with sync to open file

    " map nerdtree to the ctrl+n
    function MyNerdToggle()
        if &filetype == 'nerdtree' || exists("g:NERDTree") && g:NERDTree.IsOpen()
            :NERDTreeToggle
        else
            :NERDTreeFind
        endif
    endfunction
    
    nnoremap <C-n> :call MyNerdToggle()<CR>
