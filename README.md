# Проект "ИщуРаботу.ру"

### Посмотреть можно здесь:

<a href="https://user223560.github.io/vuejs-jobs-app/" target="_blank">https://user223560.github.io/vuejs-jobs-app/</a>

## Что использовалось в разработке

<p>Кодовая база - фреймворк <b>Vue.js</b>, стилизация - фреймворк <b>Tailwind CSS</b></p>
<p>Вспомогательные инструменты:</p>
<ul>
  <li><b>Vue Router</b> - навигация по страницам</li>
  <li><b>Vue Spinner</b> - анимация загрузки</li>
  <li><b>Vue Toastification</b> - окна уведомлений</li>
  <li><b>Bootstrap Icons</b> - иконки</li>
</ul>

## Функционал приложения

<p>Данный "веб-сервис для соискателей и работодателей" представляет из себя <b>SPA</b>, в котором реализована имитация работы с бэкендом. Для этого используется сочетание работы с локальным JSON-файлом и с <b><code>localStorage</code></b>.</p>
<p>Для "соискателей" предусмотрен просмотр вакансий, для "работодателей" - размещение и редактирование вакансий.</p>
<p>Функции изменения и удаления доступны только для вакансий, созданных "работодателем" (операции с <code>localStorage</code>). Вакансии, существующие "по умолчанию" (достающиеся из JSON-файла) изменять нельзя.</p>

### Прочие особенности

<ul>
  <li>Использование <code>Composition API</code></li>
  <li>Предусмотрена обработка различных ошибок, для чего создан отдельный компонент</li>
  <li>В вёрстке применяются, помимо прочего, <b>Grid</b>, <b>Flexbox</b>, семантические теги</li>
  <li>Приложение адаптировано под экраны мобильных устройств</li>
</ul>

## Customize configuration

See [Vite Configuration Reference](https://vite.dev/config/).

## Project Setup

```sh
npm install
```

### Compile and Hot-Reload for Development

```sh
npm run dev
```

### Compile and Minify for Production

```sh
npm run build
```
