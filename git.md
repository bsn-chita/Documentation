Добавляем в файл ~/.bash_aliases

```bash
lazy() {
    local msg="${1:-lazy update: $(date +'%Y-%m-%d %H:%M')}"
    git pull --rebase && git add -A && git commit -m "$msg" && git push
}
```
Чтобы изменения вступили выполним

```bash
source ~/.bash_aliases
```

