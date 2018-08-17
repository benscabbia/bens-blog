Welcome to my personal blog, which you can view here: https://benscabbia.co.uk

## Development

There are two branches:

- `master` for development.  **All pull requests should be to submitted against `master`.**
- `gh-pages` for the hosted site

You can quickly setup your development environment via Docker. Simply navigate to the root project directory and run: 

```
docker-compose up
```

This will start the webserver and enable you to browse the site at: http://0.0.0.0:4000. You can then in another terminal run a second docker command to watch the directory, meaning it will detect changes and automatically recompile: 

```
docker-compose exec site jekyll build --watch
```

## Built with Jekyll > Lanyon
The blog is built with Jekyll[Jekyll](http://jekyllrb.com). The theme is forked from Lanyon:
Lanyon - an unassuming [Jekyll](http://jekyllrb.com) theme that places content first by tucking away navigation in a hidden drawer. It's based on [Poole](http://getpoole.com), the Jekyll butler.


### Themes

There are eight themes available: 

![Available theme classes](https://f.cloud.github.com/assets/98681/1817044/e5b0ec06-6f68-11e3-83d7-acd1942797a1.png)

To use a theme, add any one of the available theme classes to the `<body>` element in the `default.html` layout, like so:

```html
<body class="theme-base-08">
  ...
</body>
```

To create your own theme, look to the Themes section of [included CSS file](https://github.com/poole/lanyon/blob/master/public/css/lanyon.css). Copy any existing theme (they're only a few lines of CSS), rename it, and change the provided colors.


## License

Open sourced under the [MIT license](LICENSE.md).

<3
