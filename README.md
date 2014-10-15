##  This the repository for my blog.

Instead of writing about life in general, I will use the README to give an outline of how to arrive at a result similar to mine. 

### Set up a github pages project page:

Following [this](http://www.thinkful.com/learn/a-guide-to-using-github-pages/start/new-project/project-page/) gh-pages tutorial

* Create repo, and initialize with README (e.g. create `/github.com/nikos-daniilidis/blog`)
* Clone the repository. In the root of the directory where you want your local copy to be, do: 

		git clone https://github.com/nikos-daniilidis/blog.git

* Create an orphan gh-pages branch

		git checkout --orphan gh-pages
		git commmit -m "first commit"
		git push origin gh-pages
_Done!_

### Installing jekyll

This is one way that works ([source](http://michaelchelen.net/81fa/install-jekyll-2-ubuntu-14-04/)). Others are  possible.

* Install Ruby, Ruby development libraries, make command 

		sudo apt-get install ruby ruby-dev make

* Install jekyll 

		sudo gem install jekyll

* Circumvent current javascript issue with Ubuntu (12.04, 14.04) 

		sudo apt-get install nodejs

* `sudo gem install rdiscount --no-rdoc --no-ri`

* As per Github instructions, it is safest to use the github pages gem. Create a file called Gemfile inside your local repository. The contents of Gemfile should be:

		source 'https://rubygems.org'
		require 'json'
		require 'open-uri'
		versions = JSON.parse(open('https://pages.github.com/versions.json').read)
		gem 'github-pages', versions['github-pages']

* Install bundler in your local repository: 

		nikos@nikos-pseudomac:~/bin/github-pages/blog$ gem install bundler

* Install jekyll using bundle 

		nikos@nikos-pseudomac:~/bin/github-pages/blog$ bundle install

### After jekyll is installed, getting started


* Start new jekyll project in the location where your local gh-pages directory is

		jekyll new ~/bin/github-pages/blog/

If a warning appears, remove all files from `/blog` and repeat (I had to remove `Gemfile, Gemfile.lock`)

* Start serving an automatically generated blog on localhost:4000

		jekyll serve --watch --baseurl ''

You should be seeing a decent page already.

### Start formatting your blog

In `css/main.scss` select your css settings.

In `_config.yml` 
	* add github un-overridable defaults (lines 1-3)
	* change preexisting site settings. Important is `baseurl: "/blog"`
	* add linked_in
	* add names to exclude

In `_includes/footer.html`
	* insert section about linked_in
	* remove section about `site.title`

In `_layout/default.html`
	* insert javascript MathJax secction (latex converter for markdown)

In `about.md`
	* insert your text
	* make sure the links to all files have the `site.url` variable inserted: `[get the PDF]({{ site.url }}/assets/mydoc.pdf)`

In all contents of `_posts`, `publications`, etc. make sure `assets` path rule is also followed.

That's it!

### Combinations of paths in `_config.yml`

| code | basseurl | url | page links | file links | broken |
|------|----------|-----|------------|------------|--------|
| 000  | blog | http://nikos-daniilidis.github.io | http://nikos-daniilidis.github.io/blog/blog/about/ | http://nikos-daniilidis/github.io/assets/all-pdf/daniilidis-resume.pdf | all links |
| 001  | blog | http://nikos-daniilidis.github.io/blog | http://nikos-daniilidis.github.io/blog/blog/about/ | http://nikos-daniilidis/github.io/assets/all-pdf/daniilidis-resume.pdf |
| 010  | blog/ | http://nikos-daniilidis.github.io |  |  |  |
| 011  | blog/ | http://nikos-daniilidis.github.io/blog |  |  |  |
| 100  | /blog | http://nikos-daniilidis.github.io |  |  |  |
| 101  | /blog | http://nikos-daniilidis.github.io/blog |  | http://nikos-daniilidis/github.io/blog/assets/all-pdf/daniilidis-resume.pdf | file links |
| 111  | /blog/ | http://nikos-daniilidis.github.io/blog |  |  | file links |



