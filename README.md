# Useful Commands
Resolve dependencies
```bash
bundle
```

Serve the website
```bash
bundle exec jekyll s
#if there is an error about the lock file, remove it and run the command above:
rm Gemfile.lock && bundle exec jekyll s
```