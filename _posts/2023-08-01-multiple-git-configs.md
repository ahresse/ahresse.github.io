---
layout: post
title:  "Multiple git configurations on your system"
date:   2023-08-01 10:00:00 +0200
categories: git
---

Since I now use my personnal computer also for my professionnal activity: I wanted to use two e-mail adresses on the same system: One for my personal activities and an another one dedicated to my company activities.

Since [git 2.13][git-2.13-changelog] , it is possible to get multiple email addresses on the same system depending on the folder location by using conditional includes.

For example:

If your global config `~/.gitconfig` contains something like:

{% highlight gitconfig %}
[user]
    name = John Doe
    email = john.doe@personal.net

[includeIf "gitdir:~/company_related_folder/"]
    path = ~/company_related_folder/.gitconfig
{% endhighlight %}

With your work specific configuration `~/company_related_folder/.gitconfig` as:

{% highlight gitconfig %}
[user]
    email = john.doe@company.com
{% endhighlight %}

*Remember that `[includeIf...]` should follows default `[user]` at the top and gitdir should include the last `/`.*

Thus, git repositories that get cloned onto the `~/company_related_folder/` directory with use the company e-mail adress.

[git-2.13-changelog]: https://lwn.net/Articles/722364/
