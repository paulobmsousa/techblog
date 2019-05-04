---
layout: post
title:  "Hello, Jekyll"
date:   2019-04-30 09:38:58 -0400
categories: personal
permalink: "/hello_jekyll/"
---

![DevCorner](https://raw.githubusercontent.com/paulobmsousa/paulobmsousa.github.io/master/images/devcorner1min.jpg)

# Preview

![Jekyll Logo][logo]

In this first post, I have noted that it could be a good idea to summarize how to create **posts**, using [Jekyll][jekyll-home], that, in short, is one of the simplest and powerfull tool to create blogs.

Firstly, some reasons to chose Jekyll:
- The simplicity (in general, static generators that allows to use HTML and CSS);
- No database needed (which implies that is simpler to deploy in a server - specially [GitLab pages][gitlab-pages]);
- Markdown syntax (user-friendly).

Now, let us start the procedure to the first blog...

# Basic Procedure

1. In the Jekyll site [Jekyll site][jekyll-site] follow the instructions.  

   - Install a full Ruby development environment;  

   - Install Jekyll and bundler gems;  

2. Use the following command to create your own blog: `_jekyll new blog_name`

3. Inside the blog folder, you can start the server: `bundle exec jekyll serve`

4. By default, the Jekyll server starts on socket **4000**.

# Common Problems

Of course, sometimes following the above simple steps cannot work as expected.

   - Jekyll demands Ruby and RubyGems and, of course, installation problems must be solved, before starting using Jekyll.  

   - Under Windows environment, the process for installing Ruby is better conducted by using [Ruby Installer][ruby-installer].  
   
   - Aditionally, the Jekyll site has a step-by-step procedure to help Windows users: [Jekyll on Windows][win-users].  
   
   - Note that, probably you are using the *minima* theme bundle, that can be replaced whenever you want. But it is ideal for most cases.  

# Configuration: config.yml

The **config.yml** is the main configuration file and it is loaded once (not in run-time), what means that, differently from Posts, you need to rebuild (`bundle exec jekyll build`) and restart (`bundle exec jekyll serve`).

The main sections are:

   - title
   - description
   - baseurl
   - url
   - markdown

Some references that are useful can also be declared, such as:
   - email
   - twitter_username
   - github_username
   - theme

Most are self intuitive, however, **baseurl**, **theme** and **markdown** are specially important. Let us describe:

   + **baseurl**: The relative path to the site. If it is left blanked, it will point to the root directory from the *url*. It is suggested that you define a specific *baseurl* to your blog;

   + **markdown**: The default markdown style. The option *"kramdown"* is the default for Jekyll;

   + **theme**: The style (CSS behaviour) of your blog. The default (and very common used) is *"minima"*.

   + **permalink**: The configuration of the URL shown in the Address bar. By default, it comes from a well defined structure: *"/:categories/:year/:month/:day/:title"*. If you prefer a simpler structure, you can adjust this item.

For instance, my blog has the following configuration:

{% highlight ruby %}
title: PauloBMSousa Blog
email: "paulo.beniciol@gmail.com"
description: "Personal Blog (incluing Family and Professional facts)."
baseurl: "/github-blog"
url: "https://paulobmsousa.github.io"
twitter_username: paulobmsousa
github_username:  paulobmsousa
markdown: kramdown
theme: minima
permalink: /:title
plugins:
  - jekyll-feed
comments: true
{% endhighlight %}

# Posts: General Structure

It is time to take some notes aboute the most important structure: how posts are organized.

## File structure

Internally, Jekyll uses some folders in order to generate the minimal self-contained post structure. However, sometimes, the basic configuration must be complemented.

For instance, if you type `_jekyll new myblog`, Jekylll will perform something like this:

{% highlight ruby %}
C:\PERSONAL> jekyll new myblog
Running bundle install in C:/PERSONAL/myblog...
  Bundler: Fetching gem metadata from https://rubygems.org/...........
  Bundler: Fetching gem metadata from https://rubygems.org/.
  Bundler: Resolving dependencies...
  Bundler: Using public_suffix 3.0.3
  Bundler: Using addressable 2.6.0
  Bundler: Using bundler 2.0.1
  Bundler: Using colorator 1.1.0
  Bundler: Using concurrent-ruby 1.1.5
  Bundler: Using eventmachine 1.2.7 (x64-mingw32)
  Bundler: Using http_parser.rb 0.6.0
  Bundler: Using em-websocket 0.5.1
  Bundler: Using ffi 1.10.0 (x64-mingw32)
  Bundler: Using forwardable-extended 2.6.0
  Bundler: Using i18n 0.9.5
  Bundler: Using rb-fsevent 0.10.3
  Bundler: Using rb-inotify 0.10.0
  Bundler: Using sass-listen 4.0.0
  Bundler: Using sass 3.7.4
  Bundler: Using jekyll-sass-converter 1.5.2
  Bundler: Using ruby_dep 1.5.0
  Bundler: Using listen 3.1.5
  Bundler: Using jekyll-watch 2.2.1
  Bundler: Using kramdown 1.17.0
  Bundler: Using liquid 4.0.3
  Bundler: Using mercenary 0.3.6
  Bundler: Using pathutil 0.16.2
  Bundler: Using rouge 3.3.0
  Bundler: Using safe_yaml 1.0.5
  Bundler: Using jekyll 3.8.5
  Bundler: Using jekyll-feed 0.12.1
  Bundler: Using jekyll-seo-tag 2.6.0
  Bundler: Using minima 2.5.0
  Bundler: Using tzinfo 2.0.0
  Bundler: Using tzinfo-data 1.2019.1
  Bundler: Using wdm 0.1.1
  Bundler: Bundle complete! 5 Gemfile dependencies, 32 gems now installed.
  Bundler: Use `bundle info [gemname]` to see where a bundled gem is installed.
New jekyll site installed in C:/PERSONAL/myblog.
{% endhighlight %}

In fact, these folders and files are created:

{% highlight windows %}
30/04/2019  14:52    <DIR>          .
30/04/2019  14:52    <DIR>          ..
30/04/2019  14:52                35 .gitignore
30/04/2019  14:52               398 404.html
30/04/2019  14:52               539 about.md
30/04/2019  14:52             1.069 Gemfile
30/04/2019  14:52             1.857 Gemfile.lock
30/04/2019  14:52               175 index.md
30/04/2019  14:52             1.652 _config.yml
30/04/2019  14:52    <DIR>          _posts
{% endhighlight %}

As you can see the structure is very basic and not complete. In fact, in this case, Jekyll will use aditional configuration folders in the default theme (for instance, *minima*). They are located in a proper folder, set during the bundle installation.

You can use commands in order to identify *where* a specific bundle is installed in your computer: `bundle show minima`.  For instance, in my version, (2.5.0) the minima bundle contains: 

{% highlight windows %}
04/04/2019  15:39    <DIR>          .
04/04/2019  15:39    <DIR>          ..
04/04/2019  15:39    <DIR>          assets
04/04/2019  15:39             1.115 LICENSE.txt
04/04/2019  15:39             8.848 README.md
04/04/2019  15:39    <DIR>          _includes
04/04/2019  15:39    <DIR>          _layouts
04/04/2019  15:39    <DIR>          _sass
{% endhighlight %}

  + **Tip**: It is a good idea to copy the bundle configuration folders to your own personal blog directory, if you want to make different configurations to your blog.

## Posts and Markdown

One of the main advantages of using Jekyll in order to create posts is the user-friendly language to create and update posts. 

1. Firstly, all the posts must be located inside the **_posts** directory.

2. Each one must have the following pattern in the filename: `[year]-[month]-[day]-[title].markdown`.

3. Inside, you can use an ordinary text plain file, using, basically, a **markdown** syntax. In [Posts section][jekyll-markdown] you can see some syntax fundamentals.

## Home configuration

Posts are shown according the descendent time order (most recent first). This pattern is conventional and you should respect this organization.

In order to get it, my **home.html** is set as shown below:

{% highlight ruby %}
{% raw %}
---
layout: default
---
<div class="home">
  {%- if site.posts.size > 0 -%}
    <h2 class="post-list-heading">{{ page.list_title | default: "Posts" }}</h2>
    <ul class="post-list">
      {%- for post in site.posts -%}
      <li>
        {%- assign date_format = site.minima.date_format | default: "%b %-d, %Y" -%}
        <span class="post-meta">{{ post.date | date: date_format }}</span>
        <h3>
          <a class="post-link" href="{{ post.url | relative_url }}">
            {{ post.title | escape }}
          </a>
        </h3>
        {%- if site.show_excerpts -%}
          {{ post.excerpt }}
        {%- endif -%}
      </li>
      {%- endfor -%}
    </ul>
    <p class="rss-subscribe">subscribe
      <a href="{{ "/feed.xml" | relative_url }}">via RSS</a>
    </p>
  {%- else -%}
    <h2>No posts found</h2>
  {%- endif -%}
</div>
{% endraw %}
{% endhighlight %}

Note that there is a test (*if* clause) in order to see if there is more than one post and, inside, a loop (*for* statement) that looks for post and shows according the date.

## Posts configuration

Each posts usually follow the **post.html** template. In my case, the text is organized as follows:

{% highlight ruby %}
{% raw %}
---
layout: default
---
<article class="post h-entry" itemscope itemtype="http://schema.org/BlogPosting">

  <header class="post-header">
    <h1 class="post-title p-name" 
        itemprop="name headline">{{ page.title | escape }}
    </h1>
    <p class="post-meta">
      <time class="dt-published" 
        datetime="{{ page.date | date_to_xmlschema }}" 
        itemprop="datePublished">
          {%- assign date_format = site.minima.date_format | default: "%b %-d, %Y" -%}
        <b>{{ page.date | date: date_format }}</b>
      </time>
      {%- if page.author -%}
        • <span itemprop="author" 
           itemscope itemtype="http://schema.org/Person">
            <span class="p-author h-card" itemprop="name">
             {{ page.author }}
            </span>
          </span>
      {%- endif -%}</p>
  </header>
  <div class="post-content e-content" itemprop="articleBody">
    {{ content }}
  </div>
</article>

{% endraw %}
{% endhighlight %}

### Aditional configuration

Note that we are dealing with two configurations in this page that can be used in order to improve the quality of the posts and the feedback mechanism to the readers.

#### Using the time to read

The first step is how calculate the most probable time that an ordinary reader can take in order to read the post.

It can be done according the following steps:

1. Include in the beginning of  **posts.html** an inclusion header that can be used to calculate the time: {% raw %} `{% include minutes-to-read.html %}`{% endraw %}

2. Craate the above mentioned file inside the **_includes** directory. In my case, the content is shown:

{% highlight ruby %}
{% raw %}

{% assign words = content | number_of_words %}
{% if words < 360 %}
  {% assign minutesToRead = '1 minute to read' %}
{% else %}
  {% assign timeWords = words | divided_by: 180 %}
  {% assign minutesToRead = timeWords | append: ' minutes to read' %}
{% endif %}

{% endraw %}
{% endhighlight %}

Now, include the variable that can be used in order to calculate this time. For instance, one called **minutesToRead**, in a part of the **posts.html**: {% raw %} `<i>Estimated time: {{minutesText}}</i>` {% endraw %}

#### Including Discussion Plugin

A real post requires a feedback mechanism that allows readers to comment the posts. In my case, I preferred to use [Disqus][disqus-page].

It can be done according the following steps:

1. Firstly, you need to register (open a free account) in the [Disqus][disqus-page] web-site.

2. Include in the ending of **posts.html** a script that can be copyied from Disqus, after the configuration:

{% highlight ruby %}
{% raw %}

<div id="disqus_thread"></div>
<script>
(function() { // DON'T EDIT BELOW THIS LINE
  var d = document, s = d.createElement('script');
  s.src = 'https://paulobmsousa.disqus.com/embed.js';
  s.setAttribute('data-timestamp', +new Date());
  (d.head || d.body).appendChild(s);
})();
</script>
<noscript>
  Please enable JavaScript to view the
  <a href="https://disqus.com/?ref_noscript">
    comments powered by Disqus.
  </a>
</noscript>
<a class="u-url" href="{{ page.url | relative_url }}" hidden></a>

{% endraw %}
{% endhighlight %}

# Using GitHub

Now, it is time to publish your site in a server. For simplicity, let us consider the [Github Pages][gitlab-pages].

## Git Basics

Firstly, considering that you have already your account (otherwise, you must follow the procedure in order to open it). Now, let us create your own repository in order to publish after. 

+ **Tip**: When you create a **new repository**, Github will instruct you showing some commands like these:

{% highlight git %}
...or create a new repository on the command line

echo "# techblog" >> README.md
git init
git add README.md
git commit -m "first commit"
git remote add origin https://github.com/paulobmsousa/techblog.git
git push -u origin master

...or push an existing repository from the command line

git remote add origin https://github.com/paulobmsousa/techblog.git
git push -u origin master

...or import code from another repository
{% endhighlight %}

You can initialize this repository with code from a Subversion, Mercurial, or TFS project.

Follow the steps:

1. Create a README file. For simplicity, you can do it by: `echo "# MyBlog >> README.md"`;

2. Initialize the git *inside* your blog folder: `git init`;

3. Include all the files: `git add .`;

4. Create your first commit: `git commit -m "First commit"`;

5. Mirror your local repository to the remote one, using `git remote add origin git@github.com:[git_username]/[blogfolder]`;

6. Publish your posts: `git push -u origin master`.

7. Note that we are still using the master trunck. In a future, it is recommended to use a specific branch. In this step, you can create a branch: `git checkout -b [branchname]`. In this case, *branchname* can be, for instance, **gh-pages** (alusion to github pages);

8. Copy the entire content of the master to the new branch: `git push origin [branchname]`.

Remember, whenever it is necessary, you must update your repository, by using `git commit -m [comment]`. And aditional files must be added before the commit, by using: `git add [filename]`.

## GitHub Pages Configuration

In order to have the GitHub Pages configuration, some additional steps are needed:

A. Create a branch called **gh-pages** using `git checkout -b gh-pages`;

B. Copy all the content to this new branch: `git push origin gh-pages`;

Now, you can check your Github Pages, according the default address: **[git_username].github.io/[blogname]**

# Dr. Jekyll

Ok, the minimal information to post in your own blog is done. In fact, Jekyll is far easier from building sites by using HTML or CSS. Thanks to Ruby and Gems, many facilities are already provided. They come from the [CoC (Convention over Configuration)][wikipedia-coc].

But all these things have a price: you must follow the conventions and obey the configuration. Small changes can cause big problems. 

![Hyde][jekyllhyde]

Of course, a complete course of Jekyll and its advanteges is the better way to learn and practice the use of this very powerfull (and simple) tool.

[jekyll-home]: https://jekyllrb.com/
[jekyll-site]: https://jekyllrb.com/docs/
[ruby-installer]: https://rubyinstaller.org/
[gitlab-pages]: https://about.gitlab.com/product/pages/
[win-users]: https://jekyllrb.com/docs/installation/windows/
[disqus-page]: https://disqus.com/
[jekyll-markdown]: https://jekyllrb.com/docs/posts/
[wikipedia-coc]: https://en.wikipedia.org/wiki/Convention_over_configuration

[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/

[logo]: https://jekyllrb.com/img/logo-2x.png "Jekyll Logo"
[jekyllhyde]: https://static.tvtropes.org/pmwiki/pub/images/drjekyllmrhyde_01.jpg "Dr Jekyll Hyde"
