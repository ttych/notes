---
title: git cd command aka gcd
date: 2020-12-06 15:06:20 +0100
categories: system git
---

While working on development, I need to move around files and
directories from my Unix terminal. And sometimes I feel I would need
to have a command that allow me to move inside my git repository
(relatively to the git root directory).

I called it `gcd`.

`gcd` is a cd command that :
- should be only working inside git directory
- should associate the git root directory as the root file system
- should support standard cd usage

For example :

    from a repository subdirectory,
    when I run `gcd`,
    I go back to the git root directory.

    from a repository subdirectory,
    when I run `gcd /`,
    I go back to the git root directory.

    from a repository subdirectory,
    when I run `gcd /test`,
    I move to the test directory at the root of the git repository.

Here is a proposal of a shell function that behaves this way :

{% highlight sh %}
gcd()
{
    gcd=$(git rev-parse --absolute-git-dir --is-bare-repository 2>/dev/null)
    if [ $? -ne 0 ] || [ -z "$gcd" ];  then
        echo >&2 "not inside git repository"
        return 1
    fi

    case $gcd in
        *true)
            gcd="${gcd%?true}"
            ;;
        *false)
            gcd="${gcd%/.git?false}"
            ;;
    esac

    case $1 in
        -) case "$OLDPWD" in
               "$gcd"/*) cd - ;;
               *) echo >&2 "not inside git repository"
                  return 1
                  ;;
           esac ;;
        /*) cd "$gcd/$1" ;;
        .|./*|..|../*) cd "$1" ;;
        *) cd "$gcd/$1" ;;
    esac
}
{% endhighlight %}
