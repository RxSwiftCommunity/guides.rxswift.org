[guides.rxswift.org](http://guides.rxswift.org)
===

This website has been built using [Hugo](http://gohugo.io).

## Run

First thing to do is to install Hugo, in Mac OS X is simple with brew:

```sh
$ brew install hugo
```

After that clone the repo and run:

```sh
$ hugo server --watch
```

This command will run a server and watch for changes, the website is then available at `http://127.0.0.1:1313/`.

### Contributing

To contribute please create pull-requests direct on the master branch and I will be happy to check and merge them.

### Deployment

Every single branch is deployed automatically to the correct website version, example 'cn.rxswift.org' will re-generate the content for the Chinese version.