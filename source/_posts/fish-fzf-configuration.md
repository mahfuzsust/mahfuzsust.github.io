---
title: Fish FZF Configuration
tags:
  - fish
  - fzf
categories:
  - Tools
---

FZF is an interactive Unix filter for command-line that can be used with any list; files, command history, processes, hostnames, bookmarks, git commits, etc.

## Tree

Display directories as trees (with optional color/HTML output)

```
brew install tree
```

## fd

Simple, fast and user-friendly alternative to find.

```
brew install fd
```

## ripgrep

ripgrep is a line-oriented search tool that recursively searches the current directory for a regex pattern.

```
brew install ripgrep
```

## fish

User-friendly command-line shell for UNIX-like operating systems.

```
brew install fish
```

## fzf

fzf is a general-purpose command-line fuzzy finder.

```
brew install fzf

# To install useful key bindings and fuzzy completion:
$(brew --prefix)/opt/fzf/install
```

## FZF preview

Let's create a function that will preview file or directory using mentioned tools

```
# .config/fish/functions/__fzf_preview.fish

function __fzf_preview
    if file --mime "$argv" | grep -q directory
        tree -L 3 "$argv"
    else if file --mime "$argv" | grep -q binary
        echo "$argv is a binary file"
    else
        if command --quiet --search bat
            bat --color=always --line-range :250 "$argv"
        else if command --quiet --search cat
            cat "$argv" | head -250
        else
            head -250 "$argv"
        end
    end
end
```

Now change the default option

```
set -U FZF_DEFAULT_OPTS " \
         --multi --cycle --keep-right -1 \
         --height=80% --layout=reverse --info=default \
         --preview-window right:50%:wrap \
         --preview '__fzf_preview {}' \
         --ansi"
```

## Search

Let's make our search faster using fd and ripgrep

```
set -U FZF_ALT_C_COMMAND "fd -t d . \$dir"
set -U FZF_CTRL_T_COMMAND "rg --files --no-require-git"
```
