# vue-cli-algolia

> A Vue.js project

Test project using Algolia JS API for searching events calendar.

## Summary:

Using vue-cli's webpack-simple setup (VueJS 2).
`searchFrom` and `searchTo` are bound to start and end dates using unixtimestamp.

To do:

- ~~Use Moment.js to convert unixtimestamp to human readable dates (new Vue methods).~~ ✔
- ~~Use [Pikaday](https://github.com/dbushell/Pikaday) datepicker.~~ ✔
- ~~Add filters for event types when available in JSON feed.~~ ✔
- ~~Hook up text input~~ and add debounce.

## Build Setup

```
bash
# install dependencies
npm install

# serve with hot reload at localhost:8080
npm run dev

# build for production with minification
npm run build
```

For detailed explanation on how things work, consult the [docs for vue-loader](http://vuejs.github.io/vue-loader).
