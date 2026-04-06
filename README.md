# Introduction

This repository is to serve as both my personal blogging site as well as a
portfolio showcase.

## Configuring environment

In the case your development environment changes like mine has between macOS
and Windows. Editing the gemfile and commenting the `gem jekyll` line will
remove the dependency error with different ruby versions. Remember to run
`bundle install` after commenting.

## Testing Locally

Use the following command to test the site locally:

```Shell
bundle exec jekyll serve --baseurl=""
```

The site should be available [here](http://localhost:4000)

## Next Steps

- [ ] Use discussions to create comment section on blog posts
- [ ] Add pagnation to posts to enable longer tutorials
