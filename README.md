# Домашнее задание к занятию "09.03 CI\CD"

## Подготовка к выполнению

1. Создайте два VM в Yandex Cloud с параметрами: 2CPU 4RAM Centos7 (остальное по минимальным требованиям).
2. Пропишите в [inventory](https://github.com/netology-code/mnt-homeworks/blob/MNT-video/09-ci-03-cicd/infrastructure/inventory/cicd/hosts.yml) [playbook](https://github.com/netology-code/mnt-homeworks/blob/MNT-video/09-ci-03-cicd/infrastructure/site.yml) созданные хосты.
3. Добавьте в [files](https://github.com/netology-code/mnt-homeworks/blob/MNT-video/09-ci-03-cicd/infrastructure/files) файл со своим публичным ключом (id_rsa.pub). Если ключ называется иначе — найдите таску в плейбуке, которая использует id_rsa.pub имя, и исправьте на своё.
4. Запустите playbook, ожидайте успешного завершения.
5. Проверьте готовность SonarQube через [браузер](http://localhost:9000/).
6. Зайдите под admin\admin, поменяйте пароль на свой.
7. Проверьте готовность Nexus через [бразуер](http://localhost:8081/).
8. Подключитесь под admin\admin123, поменяйте пароль, сохраните анонимный доступ.

---

1. Создал директорию `infrastructure/terraform`, прописал настройки провайдера YC, виртуальных машин в `main.tf`, а так же вывод внешних IP адресов в и `output.tf`. Для доступа к ВМ по ssh создал пользователя и указал публичный ключ в `meta.txt` в виде:

```yml
#cloud-config
users:
  - name: azamat
    groups: sudo
    shell: /bin/bash
    sudo: ['ALL=(ALL) NOPASSWD:ALL']
    ssh-authorized-keys:
      - "ssh-ed25519 <key> dengiekk@gmail.ru"
```
Применил конфигурацию:
```bash
$ terraform apply
Apply complete! Resources: 4 added, 0 changed, 0 destroyed.

Outputs:

external_ip_nexus01 = "62.84.124.14"
external_ip_sonar01 = "62.84.124.162"
```
2. Полученные IP адреса прописал в `infrastructure/inventory/cicd/hosts.yml`.
3. Добавил `id_ed25519.pub` в `infrastructure/files`.
4. Запустил плейбук:

```sh
$ ansible-playbook -i inventory/cicd/ site.yml -vv
PLAY RECAP ************************************************************************************************************************************
nexus-01                   : ok=17   changed=15   unreachable=0    failed=0    skipped=2    rescued=0    ignored=0   
sonar-01                   : ok=35   changed=27   unreachable=0    failed=0    skipped=0    rescued=0    ignored=0 
```

5. Проверил доступность сервисов, поменял пароли.

## Знакомоство с SonarQube

### Основная часть

1. Создаём новый проект, название произвольное
2. Скачиваем пакет sonar-scanner, который нам предлагает скачать сам sonarqube
3. Делаем так, чтобы binary был доступен через вызов в shell (или меняем переменную PATH или любой другой удобный вам способ)

---

Разархивировал sonarqube, запустил в директории bin команду

```sh
$ export PATH=$PATH:$(pwd)
```

4. Проверяем `sonar-scanner --version`

```sh
$ sonar-scanner --version
INFO: Scanner configuration file: /home/azamat/Downloads/sonar-scanner-4.6.2.2472-linux/conf/sonar-scanner.properties
INFO: Project root configuration file: NONE
INFO: SonarScanner 4.6.2.2472
INFO: Java 11.0.11 AdoptOpenJDK (64-bit)
INFO: Linux 5.11.0-43-generic amd64
```

5. Запускаем анализатор против кода из директории [example](./example) с дополнительным ключом `-Dsonar.coverage.exclusions=fail.py`

```sh
sonar-scanner \
  -Dsonar.projectKey=my_project_01 \
  -Dsonar.sources=. \
  -Dsonar.host.url=http://62.84.124.162:9000 \
  -Dsonar.login=2e2516b60a03b77e516a30c2614e52ae49149dd1 \
  -Dsonar.coverage.exclusions=fail.py
```

6. Смотрим результат в интерфейсе

![Скриншот](./screenshots/1.png)

7. Исправляем ошибки, которые он выявил(включая warnings)

8. Запускаем анализатор повторно - проверяем, что QG пройдены успешно

9. Делаем скриншот успешного прохождения анализа, прикладываем к решению ДЗ

![Скриншот](./screenshots/2.png)


## Знакомство с Nexus
 
### Основная часть

1. В репозиторий `maven-public` загружаем артефакт с GAV параметрами:
   1. groupId: netology
   2. artifactId: java
   3. version: 8_282
   4. classifier: distrib
   5. type: tar.gz
2. В него же загружаем такой же артефакт, но с version: 8_102
3. Проверяем, что все файлы загрузились успешно
4. В ответе присылаем файл `maven-metadata.xml` для этого артефекта

---

После загрузки в репозитроий двух артефактов`maven-metadata.xml` выглядит так:

```xml
<metadata modelVersion="1.1.0">
  <groupId>netology</groupId>
  <artifactId>java</artifactId>
  <versioning>
    <latest>8_282</latest>
    <release>8_282</release>
    <versions>
      <version>8_102</version>
      <version>8_282</version>
    </versions>
    <lastUpdated>20211221094918</lastUpdated>
  </versioning>
</metadata>
```

![Скриншот](3.png)


### Знакомство с Maven

### Подготовка к выполнению

1. Скачиваем дистрибутив с [maven](https://maven.apache.org/download.cgi)
2. Разархивируем, делаем так, чтобы binary был доступен через вызов в shell (или меняем переменную PATH или любой другой удобный вам способ)

```sh
$ sudo mkdir /usr/local/maven
cd /usr/local/maven
$ sudo wget https://dlcdn.apache.org/maven/maven-3/3.8.4/binaries/apache-maven-3.8.4-bin.zip
$ sudo unzip apache-maven-3.8.4-bin.zip
$ sudo rm apache-maven-3.8.4-bin.zip
$ sudo ln -s /usr/local/maven/apache-maven-3.8.4/bin/mvn /usr/bin/mvn
```

3. Удаляем из `apache-maven-<version>/conf/settings.xml` упоминание о правиле, отвергающем http соединение( раздел mirrors->id: my-repository-http-unblocker)

Удалил участок кода:

```xml
<mirror>
      <id>maven-default-http-blocker</id>
      <mirrorOf>external:http:*</mirrorOf>
      <name>Pseudo repository to mirror external repositories in>
      <url>http://0.0.0.0/</url>
      <blocked>true</blocked>
    </mirror>
  </mirrors>
```

4. Проверяем `mvn --version`

```sh
$ mvn --version
Apache Maven 3.8.4 (9b656c72d54e5bacbed989b64718c159fe39b537)
Maven home: /home/azamat/Downloads/apache-maven-3.8.4
Java version: 11.0.13, vendor: Ubuntu, runtime: /usr/lib/jvm/java-11-openjdk-amd64
Default locale: en_US, platform encoding: UTF-8
OS name: "linux", version: "5.11.0-43-generic", arch: "amd64", family: "unix"
```

5. Забираем директорию [mvn](./mvn) с pom

### Основная часть

6. Меняем в `pom.xml` блок с зависимостями под наш артефакт из первого пункта задания для Nexus (java с версией 8_282)

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>
 
  <groupId>com.netology.app</groupId>
  <artifactId>simple-app</artifactId>
  <version>1.0-SNAPSHOT</version>
   <repositories>
    <repository>
      <id>my-repo</id>
      <name>maven-public</name>
      <url>http://62.84.124.14:8081/repository/maven-public/</url>
    </repository>
  </repositories>
  <dependencies>
    <dependency>
      <groupId>netology</groupId>
      <artifactId>java</artifactId>
      <version>8_282</version>
      <classifier>distrib</classifier>
      <type>tar.gz</type>
    </dependency>
  </dependencies>
</project>
```

7. Запускаем команду `mvn package` в директории с `pom.xml`, ожидаем успешного окончания

```bash
$ mvn package
...
[INFO] Scanning for projects...
[INFO] 
[INFO] --------------------< com.netology.app:simple-app >---------------------
[INFO] Building simple-app 1.0-SNAPSHOT
[INFO] --------------------------------[ jar ]---------------------------------
...
[INFO] Building jar: /home/azamat/devops/netology-9.3-cicd/mvn/target/simple-app-1.0-SNAPSHOT.jar
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  29.060 s
[INFO] Finished at: 2021-12-26T18:22:18+06:00
[INFO] ------------------------------------------------------------------------
```

8. Проверяем директорию `~/.m2/repository/`, находим наш артефакт

```sh
$ ls -la ~/.m2/repository/netology/java/8_282/
total 560
drwxrwxr-x 2 azamat azamat   4096 дек 26 18:21 .
drwxrwxr-x 3 azamat azamat   4096 дек 26 18:21 ..
-rw-rw-r-- 1 azamat azamat 547096 дек 26 18:21 java-8_282-distrib.tar.gz
-rw-rw-r-- 1 azamat azamat     40 дек 26 18:21 java-8_828-distrib.tar.gz.sha1
-rw-rw-r-- 1 azamat azamat    350 дек 26 18:21 java-8_282.pom
-rw-rw-r-- 1 azamat azamat     40 дек 26 18:21 java-8_282.pom.sha1
-rw-rw-r-- 1 azamat azamat    200 дек 26 18:21 _remote.repositories
```

9. В ответе присылаем исправленный файл `pom.xml`