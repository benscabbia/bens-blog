Welcome to my personal blog, you can view here: https://benscabbia.co.uk.

If you find a mistake or I've said something that is wrong, please feel free to submit a pull request or raise an issue :). 

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
The blog is a fork of [Lanyon](https://github.com/poole/lanyon), a [Jekyll](http://jekyllrb.com) theme. 

> Lanyon - an unassuming [Jekyll](http://jekyllrb.com) theme that places content first by tucking away navigation in a hidden drawer. It's based on [Poole](http://getpoole.com), the Jekyll butler.

## License

Open sourced under the [MIT license](LICENSE.md).

<3
