---
layout: post
title:  "Installing Taiga Docker container on an Azure VM"
date:   2023-08-12 14:12:25 -0500
categories: azure docker taiga
---
Requirements:
- Azure Ubuntu VM created
- Public IP attached to VM

### Creating SSL cert on Azure VM
We will create the Let's Encrypt cert on the server. This will be mapped in the Docker container in the next steps.

Follow the steps [here](https://certbot.eff.org/instructions?ws=other&os=ubuntufocal) to setup certbot to handle the Let's Encrypt certificate (and automatic renewal).

### Modifying docker-compose.yml to use Let's Encrypt cert

Github repo is located [here](https://github.com/taigaio/taiga-docker), specifically the [docker-compose.yml](https://github.com/taigaio/taiga-docker/blob/master/docker-compose.yml) is important and what we will be modifying.

Changes in `docker-compose.yml`:



Jekyll requires blog post files to be named according to the following format:

`YEAR-MONTH-DAY-title.MARKUP`

Where `YEAR` is a four-digit number, `MONTH` and `DAY` are both two-digit numbers, and `MARKUP` is the file extension representing the format used in the file. After that, include the necessary front matter. Take a look at the source for this post to get an idea about how it works.

Jekyll also offers powerful support for code snippets:

{% highlight ruby %}
def print_hi(name)
  puts "Hi, #{name}"
end
print_hi('Tom')
#=> prints 'Hi, Tom' to STDOUT.
{% endhighlight %}

Check out the [Jekyll docs][jekyll-docs] for more info on how to get the most out of Jekyll. File all bugs/feature requests at [Jekyll’s GitHub repo][jekyll-gh]. If you have questions, you can ask them on [Jekyll Talk][jekyll-talk].

[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/