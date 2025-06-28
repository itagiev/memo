### General

general -> pull requests -> Allow squash merging (Pull request title) \
Оставить только одну стратегию при выполнении pull request

general -> pull requests -> Always suggest updating pull request branches (check) \
К примеру если feature branch отстает от main, тесты могут не пройти или ложно пройти и т.п.

general -> pull requests -> Allow auto-merge (check)

general -> pull requests -> Automatically delete head branches (check)

### Branches

Создать правило для ветки main

Require a pull request before merging (check)

Require a pull request before merging -> Dismiss stale pull request approvals when new commits are pushed (check) \
Если в feature ветку был запушен новый комит, отменяет approval для текущего pull request

Require a pull request before merging -> Required approvals (n - кол-во человек)

Require status checks to pass (check)

Require status checks to pass -> Require branches to be up to date before merging (check)