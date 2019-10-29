# Проверка кода 1С через SonarQube

Клонируйте текущий проект:
`git clone http://путь-к-репозиторию` (например, `git clone http://git.a/devOps/sonarqube-1c.git`)

## Требуемое программное окружение

* Git
* Docker
* Java 11

## Структура проекта проверки через SonarQube

```text
sonarqube-1c/
├── projects/
├── sonar-scanner/
├── sonar-server/
├── README.md
├── run-scanner.bat
└── sonar-project.properties
```

### Расшифровка

#### Каталоги

`./projects` - для репозиториев с проектами, которые необходимо анализировать;

`./sonar-scanner` - для хранения исполняемых файлов сканера, с помощью которого производится анализ;

`./sonar-server` - для конфигурации и сборки контейнера с сервером SonarQube;

#### Файлы

`README.md` - файл с описанием проекта в формате Markdown. Предполагается, что сначала описание подготавливается в файле README.md (мастер), а затем импортируется в confluence через "Вставить разметку -  Вставить Confluence Wiki". Преобразовывать оформление можно через [конвертер разметки](http://chunpu.github.io/markdown2confluence/browser/). "Вставить разметку - Вставить Markdown" в Confluence 6.15.6 работает некорректно, ломая разметку. В более поздних версиях ситуация может измениться.

`run-scanner.bat` - файл для запуска процесса сканирования;

`sonar-project.properties` - шаблон-заготовка с настройками SonarQube Scanner;

## Установка локального сервера SonarQube*

**При использовании корпоративного сервера GitLab установка своего сервера не требуется*.

Весь стек серверной части SonarQube разворачивается в docker-контейнерах через docker-compose. Для проверки кода по правилам 1С требуется плагин [SonarQube 1C (BSL) Community Plugin](https://1c-syntax.github.io/sonar-bsl-plugin-community/). Процесс сборки собственного локального сервера:

1. Клориновать текущий проект.

2. Скачать актуальную [версию 1C (BSL) Community Plugin](https://github.com/1c-syntax/sonar-bsl-plugin-community/releases);

3. Поместить плагин в каталог клонированного проекта: `sonar-server\sonarqube\plugins`;

4. Через CLI перейти в корень проекта и выполнить: `docker-compose -f "sonar-server\docker-compose.yml" up -d --build`;

5. Проверить запустился ли сервер, введя в браузере <http://localhost:9000> (логин и пароль первого входа: `admin`);

**Пункты 2 и 3 необязательны, так как плагин можно установить интерактивно через "Marketplace (Магазин)", см. <http://localhost:9000/admin/marketplace>*.

## Установка сканера для SonarQube

Для того, чтобы просканировать проект 1С необходимо:

1. Клонировать текущий проект.

2. Скачать сканер: `https://docs.sonarqube.org/latest/analysis/scan/sonarscanner/`.

3. Распаковать сканер в каталог: `sonar-scanner`.

4. Для корпоративного сервера SonarQube необходимо получить token у администратора.

5. В каталог `projects` скопировать локальный репозиторий, который необходимо просканировать (или клонировать его из удаленного репозитория).

6. В корне клонированного репозитория уже может находиться файл настройки сканирования `sonar-project.properties`. Если этого файла нет, то скопируйте его из корня основного проекта и, при необходимости, измените следующие настройки:

    ```bash
    # адрес сервера SonarQube, куда необходимо отправлять проанализированные данные
    sonar.host.url=...

    # ключ проекта
    sonar.projectKey=...

    # путь к проверяемым исходным кодам, например, sonar.sources=./src или sonar.sources=./src/config
    sonar.sources=...
    ```

7. Через CLI перейти в каталог с проверяемым репозиторием и сделать checkout в ту ветку, которую необходимо проверить, например, `git checout develop`;

8. В командной строке перейти в корень основного проекта и запустить проверку, переопределив при необходимости параметры сканирования, например, `run-scanner -D"sonar.host.url=http://sonar.server" -D"sonar.projectBaseDir=./projects/gitlab-services" -D"sonar.projectVersion=1.4102" -D"sonar.login=de410ad1d044086ccf6413e32e05317c24a85a13"`

где:

* `sonar.projectBaseDir` - относительный путь к файлу настроек сканера для проекта;
* `sonar.projectVersion` - версия проекта;
* `sonar.login` - token доступа к учетной записи с правом анализировать проект;

## Возможные проблемы

Во время запуска сканирования:

```bash
ERROR: Error during SonarQube Scanner execution
ERROR: null
ERROR: Caused by: GC overhead limit exceeded
```

В файле-запуска сканирования `run-scanner` установите требуемый размер оперативной памяти: `set SONAR_SCANNER_OPTS=-Xmx6g`.

### Полезные ссылки

[Управляй качеством кода 1С с помощью SonarQube](https://infostart.ru/public/1089670/)

#### * Текущий проект проверялся на SonarQube 7.9.0.26994 и sonar-communitybsl-plugin-1.1.1.jar
