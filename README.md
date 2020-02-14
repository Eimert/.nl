# .nl
Eimert's personal blog running on GitHub pages.
- [Minimal Mistakes theme](https://github.com/mmistakes/minimal-mistakes)
- [documentation](https://mmistakes.github.io/minimal-mistakes/docs/quick-start-guide/)<br>
- [weiyangtham.github.io](https://github.com/weiyangtham/weiyangtham.github.io)<br>



## Ruby gems local setup
```bash
sudo apt-add-repository ppa:brightbox/ruby-ng
sudo apt-get update
sudo gem install mdl
bundle
```

## Lint
```bash
POSTS=~/dev/.nl/_posts/*.md
mdl $POSTS
find $POSTS -name '*_*'
```

## Local launch
```bash
jekyll serve

or:
jekyll serve --incremental
```

## Maintenance
Find out vulnerabilities by running Lighthouse (Google Chrome extension).
```bash
bundle outdated
sudo bundle update
```
Update the minimal mistakes theme in [_config.yml](_config.yml). The latest release can be found [here](https://mmistakes.github.io/minimal-mistakes/).

## Links
https://thoughtbot.com/blog/keep-your-gems-up-to-date
