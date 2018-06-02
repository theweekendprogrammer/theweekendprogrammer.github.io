---
layout: post
title: Had some fun piecing together my own oh-my-zsh theme
tags: [oh my zsh]
---

![zsh theme example](images/zsh-theme-example.jpg)

The first dot shows the previous commands return status — green for success red
for failure.

Then I show the date date time. This is useful for scrolling back in your
terminal to see what time you were running a command.

Then I show the git branch information. A red dot is show after the branch name
if there are any changes.

And here is the code:

{% raw %}
```bash
local ret_status="%(?:%{$fg_bold[green]%}● :%{$fg_bold[red]%}● )"
PROMPT='${ret_status}%{$reset_color%}%D{%H:%M:%S} %{$fg[cyan]%}%~%{$reset_color%} $(git_prompt_info)
$ '

ZSH_THEME_GIT_PROMPT_PREFIX="["
ZSH_THEME_GIT_PROMPT_SUFFIX="%{$reset_color%}"
ZSH_THEME_GIT_PROMPT_DIRTY="%{$fg[red]%}•%{$reset_color%}]"
ZSH_THEME_GIT_PROMPT_CLEAN="]"
"]"
```
{% endraw %}

