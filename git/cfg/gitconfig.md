### Авто добавление ветки в remote и установка на ней upstream

```bash
git config --global --add --bool push.autoSetupRemote true
```

```ini
[push]
	autoSetupRemote = true
```

### Создание алиасов

```bash
git config --global alias.co "checkout"
# или сложнее
git config --global alias.copm '!git checkout main && git pull'
```

```ini
a = add -A
s = status
as = !git add -A && git status
c = commit
cm = commit -m
ca = commit --amend --no-edit
co = checkout
b = branch
l = log --color --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr)%C(bold blue)<%an>%Creset' --abbrev-commit
```

### .gitconfig

```ini
[core]
	editor = code --wait
[diff]
	tool = vscode
[difftool "vscode"]
	cmd = code --wait --diff $LOCAL $REMOTE
[merge]
	tool = vscode
[mergetool "vscode"]
	cmd = code --wait $MERGED
[mergetool]
	keepBackup = false
[user]
	name = itagiev
	email = samwizardry@gmail.com
[alias]
	a = add -A
	s = status
	as = !git add -A && git status
	c = commit
	cm = commit -m
	ca = commit --amend --no-edit
	co = checkout
	b = branch
	l = log --color --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr)%C(bold blue)<%an>%Creset' --abbrev-commit
[filter "lfs"]
	clean = git-lfs clean -- %f
	smudge = git-lfs smudge -- %f
	process = git-lfs filter-process
	required = true
[push]
	autoSetupRemote = true
```