# Tmux

## options

options define behavior of tmux as a whole. These are used when starting tmux.

options list:
- -f file: you can set another config file to check when its starting.
- -L socket-name: tmux stores

## Default keybindings
C-b prefix key
C-o rotate the panes
C-z suspend the tmux client
`#` List all paste buffers
$ rename the current session
& kill the current window
' prompt for window index to select
( ) to switch between sessions
: tmux command prompt
= list the past buffers
? list all keybindings
D choose a client to detach
`[` enter copy mode
`]` paste copied buffer
c create a new window
d detach the current client
m mark the pane
M unmark the pane
n next window
p previous window
s choose the session
x kill the pane
M-n Move to the next window with a bell or activity marker.
M-o Rotate the panes in the current window backwards.
M-p Move to the previous window with a bell or activity marker.

## Command 
commands can control tmux behavior and have many flag options to be seted.
commands can run using configs, shell, bind-key, if-shell or confirm-before.
### clients and sessions
### windows and panes

## Aliases
each command of tmux has an alias.

## Env variables:  
- TMUX_TMPDIR: path where all tmux servers are stored, by default its /tmp (it gets deleted at the time of reboot).

