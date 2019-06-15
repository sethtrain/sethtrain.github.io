Title: Decent Python workflow in Vim
Date: 2019-06-14 20:00
Category: Development
Tags: python,vim,tmux,lsp

Like many others, I've been using IntelliJ's Python plugin to do Python development.  One would be hardpressed to find
a better environment to develop within.  I've often told people that Ideavim is the best Vim I've ever used.  Not because
it is more powerful than Vim proper but because of the ecosystem (IntelliJ) from which Ideavim runs.

In recent years, we've seen tools like Microsoft's Visual Studio Code become more and more useful and powerful.  It feels like
with the introduction of these new tools we seen the introduction of supporting tool, such as language servers, come to light.
While these systems aren't new I've never used them until recently and I have been pleasantly surprised with the power they provide.

All of this to say, I still find Vim the place I want to work.  Other vim emulators are decent but their is still a plugin
system within the Vim ecosystem that I find valuable in my workflows and wanted to see if I could come up with an acceptable
alternative.  Let's take a ride and see where we stop...

Disclaimer
----------

Software engineers are quite opinionated about their development environments, me included.  Hopefully this article and tools
mentioned within gives you the ability to choose the bits you like and lets you throw away and replace the items you don't.
My workflow is just that, my workflow.  It isn't perfect but it is within the realm of what makes me feel comfortable. I'll
just quickly mention here some base tools that I use to make this workflow happen.  I use `pyenv`, `pipenv`, `vim` (version 8)
and `tmux` with `tmuxp` to quickly and easily get me in the zone.


My goal
-------

I feel like there are a couple different tasks that make IntelliJ an optimal environment for Python.  The indexing used within
the tool provides the ability for us to jump to various places within our code whenever we want.  We can actually hop into Flask
and see the implementation of a `Blueprint`.  This is a powerful ability and the first on the list to be able to replicate.  The
next was the ability to rename functions and variables and that rename successfully propagate throughout the application.  All
the other pieces of functionality can be wrapped up in various tools, as you will see, so I won't make those a part of my initial
goal.

Vim
---

I've found these plugins are the ideal plugins to lay the foundation for my main goals.  I use `vim-plug` as my plugin manager
but you should be able to use most of the current plugin managers.

```vimscript
Plug 'prabirshrestha/async.vim'
Plug 'prabirshrestha/vim-lsp'
Plug 'w0rp/ale'
Plug 'davidhalter/jedi-vim'

" ------------------------------------------------------------------------------
" LSP
" ------------------------------------------------------------------------------
if executable('pyls')
    au User lsp_setup call lsp#register_server({
            \ 'name': 'pyls',
            \ 'cmd': {server_info->['pyls']},
            \ 'whitelist': ['python'],
            \ })
endif
let g:lsp_diagnostics_enabled = 0 ; turn off diagnosis because I feel w0rp/ale does a better job
```
Once you have these installed and configured you just need the `python-language-server`.

Python requirements
-------------------
```config
[[source]]
url = "https://pypi.org/simple"
verify_ssl = true
name = "pypi"

[packages]

[dev-packages]
python-language-server = "~=0.26"

[scripts]
pyls = "pyls"

[requires]
python_version = "3.7.3"

```
Now how do we put these together to accomplish our goals?

Tmux and Tmuxp
--------------

I'm definitely not here to sell you on `tmux` or `tmuxp` just show you how I use them to glue it all up.  Here is an example `.tmuxp.yml`
file:

```yml
start_directory: "${PWD}"
shell_command_before: "pyenv local 3.7.3"
session_name: project_name
environment:
  VIRTUAL_ENV: "change-me"
windows:
- window_name: vim
  panes:
      - vim
- window_name: execs
  panes:
      - pipenv run pyls
```
There is definitely one special thing to note here.  `vim-jedi` still doesn't easily work with `pipenv` so I accomplish the
loading of the `virtualenv` by setting the `VIRTUAL_ENV` environment variable.  Without it `vim-jedi` will try to find the definitions
within the system python.

Once this is established a simple `tmuxp load .` will load of the session and you are on your way.

See it in action
----------------
Let's see it in action...

<iframe src="https://player.vimeo.com/video/342394889" width="640" height="400" frameborder="0" allow="autoplay; fullscreen" allowfullscreen></iframe>
<p><a href="https://vimeo.com/342394889">vim-python-workflow</a> from <a href="https://vimeo.com/user394719">Seth Buntin</a> on <a href="https://vimeo.com">Vimeo</a>.</p>

Notable mentions
----------------

While we just went over a couple of tasks to potentially replace my IntelliJ workflow it definitely isn't everything.  Some notable mentions
are: `Tagbar`, `vim-fugitive`, `dispatch` (for running quick tasks while in the editor), `NERDTree`, `vimux` and `fzf` to name a few more.

I hope you have found these useful and now feel empowered to embark on the journey to use the tools that we have all loved for many years
now with some of the more modern abilities of today's IDE's and tools.
