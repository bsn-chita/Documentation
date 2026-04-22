Добавляем в файл ~/.bash_aliases

```bash
alias gstatus='git status'
alias glog='git log --oneline -n 10'
alias gpull='git pull --rebase'

gpush() {
    # 1. Собираем сообщение: Дата [ + Ваш комментарий, если он есть]
    local date_part="$(date +'%Y-%m-%d %H:%M')"
    local msg
    if [ -z "$1" ]; then
        msg="update: $date_part"
    else
        msg="update $date_part: $1"
    fi
    # 2. Сначала подготавливаем и сохраняем локально
    git add -A && \
    git commit -m "$msg" && \
    # 3. Только теперь синхронизируемся и отправляем
    git pull --rebase && \
    git push
}
```
Чтобы изменения вступили выполним

```bash
source ~/.bash_aliases
```




