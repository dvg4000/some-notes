---
layout: post
title:  "Some Firefox customization"
date:   2022-06-17 22:10:33 +0300
tags: [Firefox]
---

В этом посте:
- как убрать скриншот в диалоге добавления закладок (bookmarks);
- как убрать лого на пустой (новой) странице.

### Убираем навязчивый скриншот из диалога добавления закладок.

Делаем как написано в [этом
посте](https://support.mozilla.org/en-US/questions/1271537).

1. Находим директорию профиля Firefox. 
Для этого надо открыть страницу [about:support](about:support). На ней найти
пункт `Profile Directory`. Значение справа и есть нужный нам каталог.
2. Создать в директории профиля, каталог `chrome` с файлом `userChrome.css`.
3. Добавить в файл `userChrome.css` текст ниже.
```css
/*** Add/Edit Bookmarks Drop-down ***/
/* Hide Giant Thumbnail and Favicon */
#editBookmarkPanelImage, 
*|div#editBookmarkPanelFaviconContainer {
      display: none !important;
}
/* Unhide URL field */
#editBMPanel_locationRow[collapsed="true"] {
  visibility: visible !important;
}
/* Taller Folder List */
#editBMPanel_folderTree {
  min-height: 350px !important; 
}
```
4. Включаем загрузку `userChrome.css`.
Для этого нужно перейти на страницу [about:config](about:config) и выставить
`toolkit.legacyUserProfileCustomizations.stylesheets` в `true`.
Далее нужно перезапустить Firefox.

Проверял на `Firefox 101.0.1 for Manjaro Linux`.


### Убираем назойливое лого Firefox-a с пустой (новой) страницы.
[Здесь все просто](https://support.mozilla.org/en-US/questions/1316284).
Переходим в [about:config](about:config) и выставляем пункт
`browser.newtabpage.activity-stream.logowordmark.AlwaysVisible` в `false`.

