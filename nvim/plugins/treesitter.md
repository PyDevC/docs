# Nvim-treesitter
* Treesitter configurations and abstraction layer for neovim. 

## Quick start

Install the parser for your language

`:TSInstall {language}`

Example:
`:TSInstall {Python}`

To get a list of supported languages

`:TSInstallInfo`

By default, everything is disabled.
To enable supported features, put this in your `init.lua` file:

`
require'nvim-treesitter.configs'.setup {
    -- A directory to install all the parsers into.
    parser_install_dir = "/some/path/to/store/parsers",

    -- Compulsory parsers to install 
    ensure_installed = {"c"},

    -- Install parsers in ensure_installed, but synchronously 
    -- Synchronous installation means that one parser will install at a time.
    -- Others will wait for one parser to install, then another will be installed
    sync_install = false,    

    highlight = {
      -- 'false' will disable the whole extension
      enable = true,

      -- list of language that will be disabled
      disable = { "c", "rust" },

      -- Setting this to true will run ':h syntax' and tree-sitter at the same time.
      -- Set this to `true` if you depend on 'syntax' being enabled (like for indentation).
      -- Using this option may slow down your editor, and you may see some duplicate highlights.
      -- Instead of true it can also be a list of languages
      additional_vim_regex_highlighting = false,
    },
  }
  vim.opt.runtimepath:append("/some/path/to/store/parsers")
`

See **nvim-treesitter-modules** for a list of all available modules and its options.

## MODULES

nvim-treesitter provides several functionalities via modules (and submodules),
each module makes use of the query files defined for each language,

All modules are disabled by default, and some provide default keymaps.
Each module corresponds to an entry in the dictionary passed to the
'nvim-treesitter.configs.setup' function, this should be in your 'init.lua' file.

`
  require'nvim-treesitter.configs'.setup {
    -- Modules and its options go here
    highlight = { enable = true },
    incremental_selection = { enable = true },
    textobjects = { enable = true },
  }
`

All modules share some common options, like 'enable' and 'disable'.
When 'enable' is 'true' this will enable the module for all supported languages,
if you want to disable the module for some languages you can pass a list to the 'disable' option.

`
  require'nvim-treesitter.configs'.setup {
    highlight = {
      enable = true,
      disable = { "cpp", "lua" },
    },
  }
`

For more fine-grained control, 'disable' can also take a function and
whenever it returns 'true', the module is disabled for that buffer.
The function is called once when a module starts in a buffer and receives the

`
  require'nvim-treesitter.configs'.setup {
    highlight = {
      enable = true,
      disable = function(lang, bufnr) -- Disable in large C++ buffers
        return lang == "cpp" and vim.api.nvim_buf_line_count(bufnr) > 50000
      end,
    },
  }
`

Options that define or accept a keymap use the same format you use to define
keymaps in Neovim, so you can write keymaps as  `gd`, `<space>a`, `<leader>a`
`<C-a (control + a), `<A-n>` (alt + n), `<CR>` (enter), etc.

External plugins can provide their own modules with their own options,
those can also be configured using the `nvim-treesitter.configs.setup`
function.


## HIGHLIGHT

Consistent syntax highlighting.

Query files: `highlights.scm`.
Supported options:

- enable: `true` or `false`.
- disable: list of languages.
- additional_vim_regex_highlighting: `true` or `false`, or a list of languages.
  Set this to `true` if you depend on 'syntax' being enabled
  (like for indentation). Using this option may slow down your editor,
  and you may see some duplicate highlights.
  Defaults to `false`.

`
  require'nvim-treesitter.configs'.setup {
    highlight = {
      enable = true,
      custom_captures = {
        -- Highlight the @foo.bar capture group with the "Identifier" highlight group.
        ["foo.bar"] = "Identifier",
      },
      -- Setting this to true or a list of languages will run `:h syntax` and tree-sitter at the same time.
      additional_vim_regex_highlighting = false,
    },
  }
`

You can also set custom highlight captures
`
  lua <<EOF
    require"nvim-treesitter.highlight".set_custom_captures {
      -- Highlight the @foo.bar capture group with the "Identifier" highlight group.
      ["foo.bar"] = "Identifier",
    }
  EOF
`
Note: The api is not stable yet.


## INCREMENTAL SELECTION

Incremental selection based on the named nodes from the grammar.

Query files: `locals.scm`.
Supported options:
- enable: `true` or `false`.
- disable: list of languages.
- keymaps:
  - init_selection: in normal mode, start incremental selection.
    Defaults to `gnn`.
  - node_incremental: in visual mode, increment to the upper named parent.
    Defaults to `grn`.
  - scope_incremental: in visual mode, increment to the upper scope
    (as defined in `locals.scm`). Defaults to `grc`.
  - node_decremental: in visual mode, decrement to the previous named node.
    Defaults to `grm`.

`
  require'nvim-treesitter.configs'.setup {
    incremental_selection = {
      enable = true,
      keymaps = {
        init_selection = "gnn",
        node_incremental = "grn",
        scope_incremental = "grc",
        node_decremental = "grm",
      },
    },
  }
`


## INDENTATION

Indentation based on treesitter for the |=| operator.
NOTE: this is an experimental feature.

Query files: `indents.scm`.
Supported options:
- enable: `true` or `false`.
- disable: list of languages.
`
  require'nvim-treesitter.configs'.setup {
    indent = {
      enable = true
    },
  }
`

`@indent`				    *nvim-treesitter-indentation-queries*
Queries can use the following captures: `@indent.begin` and `@indent.dedent`,
`@indent.branch`, `@indent.end` or `@indent.align`. An `@indent.ignore` capture tells
treesitter to ignore indentation and a `@indent.zero` capture sets
the indentation to 0.

`@indent.begin`			    *nvim-treesitter-indentation-indent.begin*
The `@indent.begin` specifies that the next line should be indented.  Multiple
indents on the same line get collapsed. Eg.

`
  (
   (if_statement)
   (ERROR "else") @indent.begin
  )
`
Indent can also have `indent.immediate` set using a `#set!` directive, which
permits the next line to indent even when the block intended to be indented
has no content yet, improving interactive typing.

eg for python:
`
 ((if_statement) @indent.begin
   (#set! indent.immediate 1))
`

Will allow:
`
  if True:<CR>
      # Auto indent to here
`

`@indent.end`				*nvim-treesitter-indentation-indent.end*
An `@indent.end` capture is used to specify that the indented region ends and
any text subsequent to the capture should be dedented.  

`@indent.branch`		    *nvim-treesitter-indentation-indent.branch*
An `@indent.branch` capture is used to specify that a dedented region starts
at the line including the captured nodes.

`@indent.dedent`		    *nvim-treesitter-indentation-indent.dedent*
A `@indent.dedent` capture specifies dedenting starting on the next line.
>
`@indent.align`		    *nvim-treesitter-indentation-aligned_indent.align*
Aligned indent blocks may be specified with the `@indent.align` capture.
This permits

>
  foo(a,
      b,
      c)
<
As well as
>
  foo(
    a,
    b,
    c)
<
and finally
>
  foo(
    a,
    b,
    c
  )
<
To specify the delimiters to use `indent.open_delimiter` and
`indent.close_delimiter` should be used.  Eg.
>
 ((argument_list) @indent.align
  (#set! indent.open_delimiter "(")
  (#set! indent.close_delimiter ")"))
<

For some languages the last line of an `indent.align` block must not be
the same indent as the natural next line.

For example in python:

>
  if (a > b and
      c < d):
      pass

Is not correct, whereas
>
  if (a > b and
	c < d):
      pass

Would be correctly indented.  This behavior may be chosen using
`indent.avoid_last_matching_next`.  Eg.

>
 (if_statement
  condition: (parenthesized_expression) @indent.align
  (#set! indent.open_delimiter "(")
  (#set! indent.close_delimiter ")")
  (#set! indent.avoid_last_matching_next 1)
 )
<
Could be used to specify that the last line of an `@indent.align` capture
should be additionally indented to avoid clashing with the indent of the first
line of the block inside an if.

==============================================================================
COMMANDS                                            *nvim-treesitter-commands*

								  *:TSInstall*
:TSInstall {language} ...~

Install one or more treesitter parsers.
You can use |:TSInstall| `all` to install all parsers. Use |:TSInstall!| to
force the reinstallation of already installed parsers.
							      *:TSInstallSync*
:TSInstallSync {language} ...~

Perform the |:TSInstall| operation synchronously.

							      *:TSInstallInfo*
:TSInstallInfo~

List information about currently installed parsers

								   *:TSUpdate*
:TSUpdate {language} ...~

Update the installed parser for one more {language} or all installed parsers
if {language} is omitted. The specified parser is installed if it is not already
installed.

							       *:TSUpdateSync*
:TSUpdateSync {language} ...~

Perform the |:TSUpdate| operation synchronously.

								*:TSUninstall*
:TSUninstall {language} ...~

Deletes the parser for one or more {language}. You can use 'all' for language
to uninstall all parsers.

								*:TSBufEnable*
:TSBufEnable {module}~

Enable {module} on the current buffer.
A list of modules can be found at |:TSModuleInfo|

							       *:TSBufDisable*
:TSBufDisable {module}~

Disable {module} on the current buffer.
A list of modules can be found at |:TSModuleInfo|

								*:TSBufToggle*
:TSBufToggle {module}~

Toggle (enable if disabled, disable if enabled) {module} on the current
buffer.
A list of modules can be found at |:TSModuleInfo|

								*:TSEnable*
:TSEnable {module} [{language}]~

Enable {module} for the session.
If {language} is specified, enable module for the session only for this
particular language.
A list of modules can be found at |:TSModuleInfo|
A list of languages can be found at |:TSInstallInfo|

							       *:TSDisable*
:TSDisable {module} [{language}]~

Disable {module} for the session.
If {language} is specified, disable module for the session only for this
particular language.
A list of modules can be found at |:TSModuleInfo|
A list of languages can be found at |:TSInstallInfo|

								*:TSToggle*
:TSToggle {module} [{language}]~

Toggle (enable if disabled, disable if enabled) {module} for the session.
If {language} is specified, toggle module for the session only for this
particular language.
A list of modules can be found at |:TSModuleInfo|
A list of languages can be found at |:TSInstallInfo|

							       *:TSModuleInfo*
:TSModuleInfo [{module}]~

List the state for the given module or all modules for the current session in
a new buffer.

These highlight groups are used by default:
>
    highlight default TSModuleInfoGood guifg=LightGreen gui=bold
    highlight default TSModuleInfoBad  guifg=Crimson
    highlight default link TSModuleInfoHeader    Type
    highlight default link TSModuleInfoNamespace Statement
    highlight default link TSModuleInfoParser    Identifier
<

								*:TSEditQuery*
:TSEditQuery {query-group} [{lang}]~

Edit the query file for a {query-group} (e.g. highlights, locals) for given
{lang}. If there are multiple files, the user is prompted to select one of them.
If no such file exists, a buffer for a new file in the user's config directory
is created. If {lang} is not specified, the language of the current buffer
is used.

						       *:TSEditQueryUserAfter*
:TSEditQueryUserAfter {query-group} [{lang}]~

Same as |:TSEditQuery| but edits a file in the `after` directory of the
user's config directory. Useful to add custom extensions for the queries
provided by a plugin.

==============================================================================
UTILS                                                  *nvim-treesitter-utils*

Nvim treesitter has some wrapper functions that you can retrieve with:
>
    local ts_utils = require 'nvim-treesitter.ts_utils'
<
Methods
						 *ts_utils.get_node_at_cursor*
get_node_at_cursor(winnr)~

`winnr` will be 0 if nil.
Returns the node under the cursor.

							  *ts_utils.is_parent*
is_parent(dest, source)~

Determines whether `dest` is a parent of `source`.
Returns a boolean.

						 *ts_utils.get_named_children*
get_named_children(node)~

Returns a table of named children of `node`.

						      *ts_utils.get_next_node*
get_next_node(node, allow_switch_parent, allow_next_parent)~

Returns the next node within the same parent.
If no node is found, returns `nil`.
If `allow_switch_parent` is true, it will allow switching parent
when the node is the last node.
If `allow_next_parent` is true, it will allow next parent if
the node is the last node and the next parent doesn't have children.

						  *ts_utils.get_previous_node*
get_previous_node(node, allow_switch_parents, allow_prev_parent)~

Returns the previous node within the same parent.
`allow_switch_parent` and `allow_prev_parent` follow the same rule
as |ts_utils.get_next_node| but if the node is the first node.

							  *ts_utils.goto_node*
goto_node(node, goto_end, avoid_set_jump)~

Sets cursor to the position of `node` in the current windows.
If `goto_end` is truthy, the cursor is set to the end the node range.
Setting `avoid_set_jump` to `true`, avoids setting the current cursor position
to the jump list.

							 *ts_utils.swap_nodes*
swap_nodes(node_or_range1, node_or_range2, bufnr, cursor_to_second)~

Swaps the nodes or ranges.
set `cursor_to_second`  to true to move the cursor to the second node

						*ts_utils.memoize_by_buf_tick*
memoize_by_buf_tick(fn, options)~

Caches the return value for a function and returns the cache value if the tick
of the buffer has not changed from the previous.

		`fn`: a function that takes any arguments
		and returns a value to store.
		`options?`: <table>
                 - `bufnr`: a function/value that extracts the bufnr from the given arguments.
                 - `key`: a function/value that extracts the cache key from the given arguments.
		`returns`: a function to call with bufnr as argument to
		retrieve the value from the cache

						  *ts_utils.node_to_lsp_range*
node_to_lsp_range(node)~

Get an lsp formatted range from a node range

							*ts_utils.node_length*
node_length(node)~

Get the byte length of node range

						   *ts_utils.update_selection*
update_selection(buf, node)~

Set the selection to the node range

						    *ts_utils.highlight_range*
highlight_range(range, buf, hl_namespace, hl_group)~

Set a highlight that spans the given range

						     *ts_utils.highlight_node*
highlight_node(node, buf, hl_namespace, hl_group)~

Set a highlight that spans the given node's range

==============================================================================
FUNCTIONS                                          *nvim-treesitter-functions*

						*nvim_treesitter#statusline()*
nvim_treesitter#statusline(opts)~

Returns a string describing the current position in the file. This
could be used as a statusline indicator.
Default options (lua syntax):
>
  {
    indicator_size = 100,
    type_patterns = {'class', 'function', 'method'},
    transform_fn = function(line, _node) return line:gsub('%s*[%[%(%{]*%s*$', '') end,
    separator = ' -> ',
    allow_duplicates = false
  }
<
- `indicator_size` - How long should the string be. If longer, it is cut from
  the beginning.
- `type_patterns` - Which node type patterns to match.
- `transform_fn` - Function used to transform the single item in line. By
  default it removes opening brackets and spaces from end. Takes two arguments:
  the text of the line in question, and the corresponding treesitter node.
- `separator` - Separator between nodes.
- `allow_duplicates` - Whether or not to remove duplicate components.

						  *nvim_treesitter#foldexpr()*
nvim_treesitter#foldexpr()~

Functions to be used to determine the fold level at a given line number.
To use it: >
  set foldmethod=expr
  set foldexpr=nvim_treesitter#foldexpr()
<

This will respect your 'foldminlines' and 'foldnestmax' settings.

Note: This is highly experimental, and folding can break on some types of
      edits. If you encounter such breakage, hitting `zx` should fix folding.
      In any case, feel free to open an issue with the reproducing steps.

==============================================================================
PERFORMANCE                                      *nvim-treesitter-performance*

`nvim-treesitter` checks the 'runtimepath' on startup in order to discover
available parsers and queries and index them. As a consequence, a very long
'runtimepath' might result in delayed startup times.


vim:tw=78:ts=8:expandtab:noet:ft=help:norl:
