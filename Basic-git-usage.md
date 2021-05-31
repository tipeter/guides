# Create a new repository on the command line

```
git config --global user.email "whoami@gmail.com"
git config --global user.name "Who am I"

touch README.md
touch .gitignore

git init
git add README.md
git add .gitignore
git commit -m "first commit"

git add -A
git commit -m "Add project files"
```

# Push an existing repository from the command line

``` 
git remote add origin https://github.com/c0ldlimit/vimcolors.git
git push origin master

git branch --set-upstream-to=origin/master master

git pull
```

Force overwriting existing files:

```
git push -f origin master
```

# Global settings for line endings

``` 
git config --global core.autocrlf input
``` 

