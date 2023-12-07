## eimertvink.nl
Eimert's personal blog running on GitHub pages using Minimal Mistakes Jekyll theme.

## Ruby gems local setup
Linux distro's with apt:
```bash
sudo apt-add-repository ppa:brightbox/ruby-ng
sudo apt-get update
gem install bundler
bundle install
```

## Mac Ruby
- [brew install chruby ruby-install](https://stackoverflow.com/questions/51126403/you-dont-have-write-permissions-for-the-library-ruby-gems-2-3-0-directory-ma)

## Lint
`mdl` Is a style checker/lint tool for markdown files.
```bash
gem install mdl
cd .nl/
POSTS=$PWD/_posts/*.md
mdl $POSTS
find $POSTS -name '*_*'
```

## Local launch
```bash
jekyll serve
# or
jekyll serve --incremental
```

## Maintenance
Update the minimal mistakes theme in [_config.yml](_config.yml) element `remote_theme`. The latest release can be found [here](https://mmistakes.github.io/minimal-mistakes/).
Find out vulnerabilities by running Lighthouse (Google Chrome extension).
```bash
gem update
bundle update --bundler
bundle outdated
bundle update
```

## Links // references
- [Minimal Mistakes theme](https://github.com/mmistakes/minimal-mistakes)
- [Minimal Mistakes documentation](https://mmistakes.github.io/minimal-mistakes/docs/quick-start-guide/)
- [Minimal Mistakes reference repo](https://github.com/weiyangtham/weiyangtham.github.io)
- [Keep your gems up to date](https://thoughtbot.com/blog/keep-your-gems-up-to-date)
