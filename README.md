# blog-source
> based on github pages & hexo

source code of my personal blog [pandalee.jp](https://pandalee.jp)

## Download
    git clone https://github.com/lihaooo/blog-source.git

## Quickly build your own hexo-github-blog
1. download this library by git clone
2. adjust some config in `[project_root]/_config.yml` and `[project_root]/_admin-config.yml`
3. `hexo new [title]` to create new article
4. deploy

## Deploy
1. finish the github pages configuration
2. modify the `deploy` config in `[project_root]/_config.yml`
3. run `hexo clean & hexo deploy`
4. input your github account & password

## Theme
1. download more themes on [hexo.io/themes](https://hexo.io/themes/index.html)
2. put the theme packages into `[project_root]/themes`
3. modify the `theme` config in `[project_root]/_config.yml`
4. rebuild & redeploy

## Common commands
1. `hexo g` build blog from `source` into `public`
2. `hexo clean` clean cache
3. `hexo deploy` deploy `public` into `github pages repo`
4. `hexo new [title]` create new article in `source`

## Hexo
more hexo usages, docs, and resources, please visit [hexo.io](https://hexo.io)

## License
[MIT License](LICENSE)