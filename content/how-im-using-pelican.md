Title: How I'm using Pelican
Date: 2019-06-15 14:15
Category: General
Tags: vim,pelican

I've obviously just recently got back into writing a blog or whatever you want to call this.  The honest reason was because I was
at home one weekend while my family was visting other family members.  Just a couple of weeks prior I had bought an
[Ergodox 76 Hotdox](https://www.alpacakeyboards.com/) which I use mainly at work.  My typing has gradually gotten better but I wanted
to use this time to continue to get better so I'm forcing myself to use it.

Pelican
-------
There are many static site generators so I did a quick search for "Python static site generator" (Python is still my go to language)
and an [article](https://www.fullstackpython.com/static-site-generator.html) that mentioned [Pelican](http://docs.getpelican.com/en/stable/)
showed up first.  I stuck with it and wanted to share how am producing this articles.

Python
------

As I mentioned in an earlier article I have `pyenv` and `pipenv` setup.  Here is my `Pipfile` for [https://sethtrain.github.io](https://sethtrain.github.io)

```config
[[source]]
url = "https://pypi.org/simple"
verify_ssl = true
name = "pypi"

[packages]
pelican = "*"
markdown = "*"

[scripts]
generate = "pelican content --output ."
preview = "pelican -l --output ."

[requires]
python_version = "3.7.3"
```

To get up and running, install these dependencies and then run `pelican-quickstart`.  It will walk you through a couple of questions
and get everything started for you. The [documentation](http://docs.getpelican.com/en/stable/quickstart.html) does a good job of
explaining all of this.

It is pretty simple.  I create a new [markdown](https://daringfireball.net/projects/markdown/) file in the `content` directory.
When I feel like I am done I will generate the site, do a quick preview locally to see if I have any issues and then push to GitHub.

Vim
---
I had established a couple of `<leader>` bindings for Pelican projects to help me generate and preview quickly.  [vim-dispatch](https://github.com/tpope/vim-dispatch)
makes this possible and easy.

```vimscript
if filereadable("pelicanconf.py")
    nmap <leader>G :Dispatch pipenv run generate<cr>
    nmap <leader>R :Dispatch pipenv run preview<cr>
endif
```

Have fun!
