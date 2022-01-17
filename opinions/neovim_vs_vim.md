# The more subtle differences between traditional Vim and Neovim

## Work in Progress

## Default Mappings

For some reason Neovim adds a few `<C-G>u` mappings to the insert mode.  To
explain these, you probably know that normally anything you do between entering
and exiting insert mode is one undo unit.  That means it takes exactly one one
`u` (or inside insert more `<C-u>`) to undo, no matter what you are writing in
this insert.  `<C-G>u` cuts this up such that `u` only undoes up to that point.
In short, it turns one undo into two undos.

The NeoVim mappings add these to `<C-W>` and to `<C-U>` by default and NeoVim
recommends unmapping them inside your `init.vim` if you don't like that.

## Double spaces after sentences

I am not quite sure yet why this is happening, but it is definitely a NeoVim
thing.

Using `J` in normal mode.
