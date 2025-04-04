# [schrodinator.github.io](https://schrodinator.github.io)

A blag.

# Setting it up

## WSL

- From Windows command prompt, `wsl --install`
	- Except I found I had already done that, apparently years ago.
	- If your WSL is as old as mine was: `wsl --update`
	- Start WSL by typing `wsl`
	- List the ancient version info of your existing Ubuntu installation with `lsb_release -a`. I found I had Ubuntu 18.-something.
- Install a recent Ubuntu image from the Microsoft Store
	- Or not-so-recent. I chose Ubuntu 22.04.5 LTS.
- Start up the distro you just installed
	- `wsl --list`
	- `wsl Ubuntu-22.04`
	- I set the new version as default: `wsl --set-default Ubuntu-22.04`
		- Next time, I need only type `wsl` to start it.
	- I deleted my old distro: `wsl --unregister Ubuntu`
- Create a new user, add it to sudoers, switch to it.
	- `adduser <username>`
		- Go through prompts to create password, etc.
	- `adduser <username> sudo` or `usermod -a -G sudo <username>`
	- `su <username>`
	- Might want to set this user as default for next time. Outside of WSL, from the Windows command prompt: `Ubuntu-22.04 config --default-user <username>`
- Realize vim is messed up. It always opens in REPLACE mode.
	- Something about stupid Windows line-ending reasons.
	- In `~/.vimrc`, add the line `set t_u7=`

## Update and install dependencies

- Update the package manager: `sudo apt update -y && sudo apt upgrade -y`
- Is `ssh` working?
	- `sudo apt install ssh`
	- `sudo service ssh restart`
- Resolve additional dependencies (a message from an alternate future in which `gem update` failed): `sudo apt install build-essential dh-autoreconf libyaml-dev openssl libssl-dev libz-dev libffi-dev`

## Git

- `git` is included but must be set up
	- `git config --global user.name <github_username>`
	- `git config --global user.email <github_email>`
	- `gh ssh-key add .ssh/id_ed25519.pub --title "WSL"` (generate an ssh key if you don't already have this)
	- Might have to generate an auth token on github.com and add it for the WSL user.
- Create a repo called `<username>.github.io`
	- I did this on github.com
	- I followed [these instructions](https://docs.github.com/en/pages/getting-started-with-github-pages/configuring-a-publishing-source-for-your-github-pages-site)
	- I named my repo `<github_username>.github.io` because that is where Github Pages will host it.
	- I then cloned the repo to WSL using `gh repo clone <github_username>/<github_username>.github.io`

## Ruby

- Install Ruby
	- `sudo apt install ruby-full`
	- Realize this version of Ruby is very old when you get a lot of warnings about incompatibilities while running `gem update`
	- `git clone https://github.com/rbenv/ruby-build.git ~/.rbenv/plugins/ruby-build`
	- `rbenv install 3.2.2`
	- Add the line `eval "$(rbenv init -)"` to your `~/.profile` ([*not* `~/.bashrc`!](https://askubuntu.com/questions/320444/setting-up-rbenv-properly)) and `source` it -- **THIS STILL DOES NOT WORK FOR ME.** I need to manually `source` the `.profile` each time I log in. Putting it in `.bashrc` didn't work either. Without doing this, when I try to preview locally using `bundle exec jekyll serve`, I get a list of packages that are required but "not" installed. They are installed, just in that specific Ruby environment that isn't automatically initialized.
	- `rbenv global 3.2.2`
	- Find the location of your OpenSSL installation: `which openssl`. Mine is /usr/bin/openssl. Use your path in the next command.
	- `gem update --with-openssl-dir=/usr/bin/openssl` (or other path to OpenSSL)

## Jekyll

- Use Ruby package manager to install Jekyll and Bundler: `gem install jekyll bundler`
- Initialize a Jekyll project in the repo
	- `cd` into the repo
	- `jekyll new --skip-bundle --force .`
	- Edit the Gemfile. Comment out (with a `#`) the line `gem "jekyll", "~> 4.3.4"`. Uncomment (remove the `#`) the line like `gem "github-pages", "~> GITHUB-PAGES-VERSION", group: :jekyll_plugins`. Replace GITHUB-PAGES-VERSION with the version number [found here](https://pages.github.com/versions/). Use the number beside `github-pages`, NOT the version beside `jekyll` or anything else!! For example, the `github-pages` version number as of this writing (2024-10-27) is 232. An integer. No minor version. So the line in my Gemfile became `gem "github-pages", "~> 232", group: :jekyll_plugins`
	- `bundle update`
	- Turns out, people seem to recommend having a very simple, two-line Gemfile for Github Pages: only the first line (`source "https://rubygems.org"`) and the line `gem "github-pages", group: :jekyll_plugins` -- without the version number. I commented out everything else in the Gemfile after the `bundle update` succeeded. I don't know if this is strictly necessary, but I wanted to avoid potential conflicts with Github Pages' installed versions of things.
- Add `Gemfile.lock` to the .gitignore

## Customize the style

- Check out the [supported themes](https://pages.github.com/themes/) for Github Pages.
- Edit `_config.yml` to reflect your chosen theme.
	- Change `theme:` in `_config.yml` to `remote-theme:`
	- Fill in your chosen theme in the format: `pages-themes/<theme>@<version>`
		- Example: For my chosen theme, [hacker](https://github.com/pages-themes/hacker/tree/master), the line becomes: `remote_theme: pages-themes/hacker@v0.2.0`
- Further customization
	- I didn't like that the default "post" layout contained "by" followed by what should have been the author's name, but was blank. I'm the only author; I don't need a "by" on every post. I also didn't like the tag separator being a single dash. I wanted an old-school comma, maybe.
	- To customize the layout, go to the Github repo of your chosen theme. Mine is [hacker](https://github.com/pages-themes/hacker/tree/master). There may be helpful instructions in the README to enable your desired customization. To change the formatting, the basic idea is: make a directory in your repo called `_layouts` and copy into it the .html file(s) from their repo that you want to change. I am okay with the main page default.html for now, but I wanted to change the formatting of posts. I copied post.html from the `hacker` repo to my own local `_layouts` directory. I then edited it to remove the "by" line and replaced the dash tag separator with a comma.

## Test

- Test locally
	- `bundle exec jekyll serve`
	- Point your browser to https://127.0.0.1:4000/. Here, you will see nothing, a blank page -- unless you have edited index.markdown to have content. This can be rather perplexing if you were expecting to see something.
- Where's my stuff?
	- Where even is the default post that Jekyll created? Ah ha, posts can usually be found at `https://127.0.0.1:4000/YYYY/MM/DD/rest-of-file-name-before-the-dot-markdown.html`
	- Somehow that's still not working; I don't see any post at that equivalent URL.
	- Edit index.markdown (could also be named index.html) to find the correct URL for us. (See my index.markdown above. I used https://github.com/mojombo/mojombo.github.io as a reference.)
	- Go back to `https://127.0.0.1:4000`and witness a link to the default Jekyll post.
	- Note that if something appears after "category:" in the post header, the URL will be `https://127.0.0.1:4000/<category>/YYYY/MM/DD/rest-of-file-name-before-the-dot-markdown.html`

## Create new content

- Edit the default Jekyll post in `_posts/`
	- Or create a new one following the file name requirements, `YYYY-MM-DD-post-title.md`
	- Restart the server (`bundle exec jekyll serve`) to see your changes.
- [This page](https://jekyllrb.com/docs/front-matter/) describes the options for "front matter" (the content between the triple dashes in the markdown header), including the predefined variables for posts.

