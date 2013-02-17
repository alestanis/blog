---
layout: post
title: "Replacing with regexes in all your code files"
date: 2013-02-17 18:39
comments: true
categories: c++ code-style perl sed scripts
---

In my first job, part of my work was to code for a big project that had started many years before, and on which many developers had worked before. *And you could see it*.

At the time, the company didn't have any global coding practices, so each one did as they pleased. A year before I came into the project, the person in charge decided developers would start using the [Google C++ Style Guide](http://google-styleguide.googlecode.com/svn/trunk/cppguide.xml), so new files looked alike and had the same feeling, but nobody did a thing for the old codebase.

At one point, I was the only developer working on the project, so I decided to unify the code with some sed-perl-regex love.

## Modifying only code files

The project was on version control, so there were lots of hidden version files with the code files. I didn't want to modify any non-code file, so I used a little `find` to work only on a subset of the files. I put everything in a script I named `sedincode`, taking a regex as a parameter:

    #!/usr/bin/env bash

	for file in $(find -name *.cpp -o -name *.h)
	do
		sed -r -i $1 $file
	done
{:lang="ruby"}

Then I started correcting some coding practices, such as adding a space after a comma

    $ ~/sedincode "s/\,([^\s])/, \1/g"
{:lang="text"}

and removing spaces before them

    $ ~/sedincode "s/([^\s])\s+\,/\1,/g"
{:lang="text"}

## Erase all the newlines!

{% img center /images/2013-02-17-erase-all-the-newlines.jpg Erase all the newlines! %}

One of the things that annoyed me the most in the codebase was having, for roughly half of the functions, opening brackets all alone in a newline, like this:

	void myFunction()
	{  // How annoying.
		cout << "Howdy!" << endl;
	}
{:lang="c++"}

To correct these, we need to span over two lines, so instead of using `sed` for my replacements, I went for `perl`, because it allows to treat the whole file at once by undefining the meaning of the end of a line with the `BEGIN{undef $/;}` snippet. I replaced `sed` by `perl` inside my `sedincode` script

	for file in $(find -name *.cpp -o -name *.h)
	do
		# sed -r -i $1 $file
		perl -i -pe "BEGIN{undef $/;} $1" $file
	done
{:lang="ruby"}

and then I used

    $ ~/sedincode "s/\n\{/ {/g"
{:lang="text"}

which simply replaces a newline followed by an opening bracket, by a space and an opening bracket. And yay, *it works!*

	void myFunction() { // Not annoying any more!
		cout << "Howdy!" << endl;
	}
{:lang="c++"}

## No! Don't erase all the newlines!

After doing some small replacements using my perl one-liner, I ran [cpplint](http://en.wikipedia.org/wiki/Cpplint) over the codebase to get a more precise feeling about other coding practices that were not put into practice and got... more than *one hundred thousand errors*.

I decided to ignore some of the errors, and try to correct others using my perl regex weapon. Many of them were about spacing in and around comments. For instance, cpplint doesn't like it when comment bars are not preceded by at least two spaces, and followed by one space:

	void myFunction() {//cpplint shows two warnings here.
{:lang="c++"}

I thought this was a piece of cake with regexes so I launched several perl replacements to fix the spacing problems around comments and punctuation.
The thing is, when you tell perl to ignore the end of the lines, sometimes it ignores *too much* of the ends of the lines (a newline can now be matched by many symbols you are *not* thinking about). I thought my replacements would behave like `sed` replacements, but instead, one of them transformed files like this:

	void myFunction() {
	//	First line of comment
	//	and the next part of the comment
		cout << "Howdy!" << endl;
	}
{:lang="c++"}

into this:

	void myFunction(){  //	First line of comment  //	and the next part of the comment
		cout << "Howdy!" << endl;
	}
{:lang="c++"}

and all the Eclipse automatic comments were broken. *Oops.*

So I instantly switched back to the `sed` version inside `sedincode`, and left the `perl` version in a comment for when I really needed it. I was also very happy of having `svn revert` that day.

## Conclusion

Perl multiline replacement (and regex replacement in general) is a strong weapon, and as such, **use it with caution** (and with some good version control).

## Bonus : some of the regexes I used

Here are some of the regexes I used to prettify the code base and correct cpplint warnings:

 - adding a space after a comma: `s/\,([^\s])/, \1/g`. You can use variations of this for other punctuation signs or operators.
 - removing spaces before references: `s/([a-zA-Z_]+)\s&\s/\1& /g`. I added an extra space to match after the `&` to avoid transforming `foo && bar` into `foo&& bar`.
 <br/>
   *See? You have to be careful, you didn't think about `&&` did you?*
 - adding two spaces before each one-line comment: `s/([^\s])\/\//\1  \/\//g` and `s/([^\s])\s\/\//\1  \/\//g` (for cases with zero and one space before the comment)
 - adding a space to transform `){` into `) {`: `s/\)\{/) {/g`
 - removing newlines before bracket openings: `s/\n\{/ {/g`. To be used with the `perl` version.

