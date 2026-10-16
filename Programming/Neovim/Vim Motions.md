Related to [[NeoVim]],
#neovim 

## Get Current File Path and Copies it
```bash
:let @+ = expand('%:p')
```

```


## Move by line number

Go to an exact line:

```vim
42G
```

That jumps to line `42`.

You can also use:

```vim
:42
```

Same result.

Go to the first line:

```vim
gg
```

Go to the last line:

```vim
G
```

## Move using relative line numbers

If `relativenumber` is enabled:

```lua
vim.opt.number = true
vim.opt.relativenumber = true
```

you will see numbers showing how far each line is from your cursor.

Example: if the target line shows `8` above your cursor:

```vim
8k
```

If the target line shows `8` below your cursor:

```vim
8j
```

## Move Cursor to the end of the line

```bash
$
```

## Move Cursor to the 'absolute' beginning of the line

```vim
0
```

```vim
^
```
Move to the first whitespace on a line
## After searching with `/`

```vim
n
```
Use to find the next match

```vim
N
```
Use to go to the previous match

## Useful line motions

Move down 5 lines:

```vim
5j
```

Move up 5 lines:

```vim
5k
```

Move to top of screen:

```vim
H
```

Move to middle of screen:

```vim
M
```

Move to bottom of screen:

```vim
L
```

 

## Practical workflow

With relative numbers enabled, you usually do this:

```text
Look at the relative number beside the target line
Press that number + j or k
```

So if the line you want is 12 lines below:

```vim
12j
```

If it is 4 lines above:

```vim
4k
```
