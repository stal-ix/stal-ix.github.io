Список зеркал исходников, из которых строится stal/IX:

https://github.com/pg83/ix/blob/main/pkgs/die/scripts/mirrors.txt

Вы можете помочь проекту, разместив у себя такое зеркало, и сделав его доступным всему миру!

Структура зеркала очень простая - это одноуровневый список файлов, и имя каждого файла является sha256sum от данных, лежащих в этом файле.

Поэтому, например, вы можете скопировать все содержимое одного из зеркал, и выставить копию в интернет.

А можете запустить готовый скрипт в cron, например, раз в час:

```
# ~/ix - clone of https://github.com/stal-ix/ix
# script will need gnu make, git, python3, wget in PATH
# output will be in ~/some/dir

cd ~/ix
git pull
./ix recache ~/some/dir
```

Инструкции для ubuntu(Катя, наверное, можно скопировать сюда, вопрос с авторством): https://gist.github.com/pg83/4bdb11a2ca3602d949db26b4b2a66781?permalink_comment_id=4687160#gistcomment-4687160

После наполнения диреткории с кешом, отправьте PR с добавлением нового зеркала в https://github.com/pg83/ix/blob/main/pkgs/die/scripts/mirrors.txt
