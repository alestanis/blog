---
layout: post
title: "Octopress and the Twilight color scheme"
date: 2013-02-04 00:17
comments: true
categories: 
---

{% img right http://ethanschoonover.com/solarized/img/solarized-yinyang.png 150 150 Solarized color schemes %}

Well, I *am* a developer (a *nazi* that-yellow-is-not-yellowy-enough developer) so the first thing I tried to do with my new blog was to see how **code looked like** in it.
Octopress uses two default themes from [Solarized](http://ethanschoonover.com/solarized); a dark one and a light one (image to the right).

I tried the dark one on a snippet of Ruby/Rails and I got this:

{% codeblock lang:ruby %}
class CookieMonster
  field :cookies, type: Integer
  
  def eat_cookies(number_of_cookies, cookie_flavor)
    if self.cookies < number_of_cookies
      raise "You don't have enough cookies!"
    end
    puts "You are going to eat #{number_of_cookies} #{cookie_flavor} cookies"
    self.cookies -= number_of_cookies
    self.save!
  end
end
{% endcodeblock %}

*Ouch!* So. Much. Blue.

When I started web development, I worked on Vim using the **Twilight color scheme**. Then, I switched to TextMate using the Twilight color scheme. Now, I work on Sublime Text 2 using... the Twilight color scheme. I **like it so much** that I even pimped my Eclipse at work (I write software in C++) for my code to look like the Twilight color scheme. The next natural step was to find a way to have code snippets inside my blog with the same colors I love and use every day.

For those of you that don't know what Twilight looks like, here's the same code snippet using its colors:

    class CookieMonster
      field :cookies, type: Integer
      
      def eat_cookies(number_of_cookies, cookie_flavor)
        if self.cookies < number_of_cookies
          raise "You don't have enough cookies!"
        end
        puts "You are going to eat #{number_of_cookies} #{cookie_flavor} cookies"
        self.cookies -= number_of_cookies
        self.save!
      end
    end
{:lang="ruby"}

Wow, I feel much better already. *Phew!*


### Getting to the dirty business

To install Twilight color scheme in Octopress, I followed what chico explained in [this blog post](http://oblita.com/blog/2012/07/06/octopress-with-mathjax-by-kramdown/), with a few tweaks.
First of all, you have to install kramdown and CodeRay by adding the following lines to your `Gemfile`:

    gem 'kramdown'
    gem 'coderay'
{:lang="ruby"}

Then, change your `_config.yml` to use kramdown:

    markdown: kramdown
    kramdown:
      use_coderay: true
      coderay:
        coderay_line_numbers: table
        coderay_css: class
{:lang="yaml"}

CodeRay accepts different line number styles: `table`, `inline`, `list` or `nil`. The difference between them is mostly about the background we use for the `CodeRay.line-numbers` class in our css: 

|                              Nil                            |                              Inline / list                        |                            Table                                |
| :---: | :---: | :---: |
| {% img /images/2013-02-04-nil.png 36 92 Nil line numbers %} | {% img /images/2013-02-04-inline.png 62 92 Inline line numbers %} | {% img /images/2013-02-04-table.png 72 92 Table line numbers %} |

<br/>

#### Tweaking the scss

To use our own color scheme, we have to add a file `sass/custom/_coderay.scss` defining the color scheme and include it by adding the following line to `sass/custom/_styles.scss`:

    @import "custom/coderay";
{:lang="text"}

TextMate's Twilight css for CodeRay is available thanks to russbrooks in [this gist](https://gist.github.com/2906599). What's great about having your own color scheme with CodeRay is that you can change around a *hundred* different colors, whereas Solarized only works with eight accent colors (you'll see later that I changed the `.CodeRay .constant` color in my custom scss file).

Next, we have to tweak the scss in order to have the right background colors and nice table line numbers. Let's add background and text colors in the `sass/custom/_colors.scss` file:

    $code-bg: #141414;
    $code-color: #F8F8F8;
    $line-nb-bg: #404040;
{:lang="text"}

Our custom scss modifications will go in the `sass/custom/_styles.scss` file. This file is read last, so anything in it will override definitions inside `_coderay.scss` (or any other scss file).

    .CodeRay {
      background-color: $code-bg;
      padding: 0px;
      pre {
        padding: 5px 0px 5px 10px;
      }
    }

    .CodeRay pre, .CodeRay .highlight, .CodeRay .gist-highlight  {
      background-color: $code-bg;
      color: #F8F8F8;
      margin-bottom: 0px;
    }

    // Nice line numbers
    table.CodeRay td {
      padding: 0px;
    }
    .CodeRay .line-numbers, .CodeRay .no {
      background-color: $line-nb-bg;
      padding-right: 1em;
      pre {
          background-color: $line-nb-bg;
          border: 0px;
          margin: 0px;
      }
    }
    .CodeRay .line { background-color: $line-nb-bg }
    .CodeRay span.line-numbers { padding: 0px 10px 0px 0px }

    // As in Sublime editor
    // Violet for constants ("Rails")
    .CodeRay .constant { color: #9B859D; }
    // Blue for predefined-constants ("self")
    .CodeRay .predefined-constant { color: #7587A6; font-weight: normal  }
{:lang="text"}

#### Adding code snippets

There are several ways to add code snippets now we have kramdown installed and customized.

##### Pygments

What's great about this color scheme customization is that you can still use the default Solarized themes if you want, as I did in the beginning of this post, with the usual `codeblock` syntax.

On the other side, you **won't be able to use the triple-backtick code blocks** any more.

##### kramdown

To add a kramdown code block, you can do the following:

        def hello
          puts "Hello!"
        end
    {:lang="ruby"}
{:lang="text"}

You can also use the single backtick syntax for inline code:

    `def hello`{:lang="ruby"}
{:lang="text"}

<br/>

And *voilà*! You have your own color scheme, customizable at will. 
