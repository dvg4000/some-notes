---
layout: post
title:  "Настройка GitHub Pages + Jekyll"
date:   2021-03-31 23:00:33 +0300
categories: jekyll github-pages
---
- Создаем репозиторий на GitHub. Можно пустой, или с Readme.md.
- Клонируем.
```sh
$ git clone ...
$ cd CLONED_REPOSITORY_DIR
```
- Настраиваем Jekyll.
[Здесь более подробно.](https://docs.github.com/en/github/working-with-github-pages/creating-a-github-pages-site-with-jekyll)
```sh
$ git checkout --orphan gh-pages
$ jekyll new .
```
- Gemfile.
```sh
$ cat Gemfile
```
```ruby
source "https://rubygems.org"
gem "minima"
gem "github-pages", group: :jekyll_plugins
group :jekyll_plugins do
   gem "jekyll-feed", "~> 0.12"
end
```
- _config.yml
```sh
$ cat _config.yml
```

  ```yml
  title: Some notes
  baseurl: "/some-notes"
  url: "https://dvg4000.github.io" 
  github_username:  dvg4000
  theme: minima
  plugins:
     - jekyll-feed
  ```

- Применяем изменения.
```sh
$ bundle update
```
- Запускаем локально, смотрим.
```sh
$ bundle exec jekyll serve --host 0.0.0.0
```
`--host 0.0.0.0` на случай, если нельзя зайти на `127.0.0.1`. Я тестирую на 
`192.168.10.10:4000/some-notes/`.
- Коммитим и пушим.
```sh
$ echo "Gimfile.lock" >> .gitignore
$ git add .
$ git commit 
$ git push origin "$(git_current_branch)"
```
