# Heroku에 서비스 배포하기 (Deploy on Heroku)

Heroku는 다양한 프로그래밍 언어와 프레임워크로 작성된 웹 애플리케이션을 지원하는 클라우드 플랫폼 서비스입니다.

이번 섹션에서는 Heroku에 vibe.d 기반으로 작성한 테스트 애플리케이션을 배포하는 과정을 설명합니다.

### 사전 준비 (Pre-requirements)

- Heroku [계정](https://signup.heroku.com/login) 이 필요합니다
- [Git](https://git-scm.com/) 이 설치되어있어야 합니다
- dmd 컴파일러 환경에서 `dub build` 명령어로 빌드했을 때, 배포하려는 애플리케이션에 이상이 없어야합니다

### 1 : 애플리케이션 환경설정 (Setup the app)

Heroku 서비스는 배포된 애플리케이션이 어떤 포트로 통신할지 알아야합니다.

이런 이유로 `PORT` 라는 운영체제 환경변수를 애플리케이션 배포에 함께 제공하고, 애플리케이션은 이 환경변수를 읽어 적절한 포트를 열어야합니다.

개발 중에는 기본 포트로 __8080__ 과 같은 포트를 설정해주는 게 권장됩니다.

```d
shared static this() {
  // ...
  auto settings = new HTTPServerSettings;
  // PORT라는 환경변수가 없을 경우를 대비해,
  // 없다면 8080을 쓰도록 지정합니다.
  settings.port = environment.get("PORT", "8080").to!ushort;
  listenHTTP(settings, router);
}
```

다음으로 `Procfile` 이라는 이름을 가진 텍스트 파일을 만듭니다. 이 텍스트 파일은 애플리케이션의 폴더 내의 최상위에 있어야하며 가동을 위해 어떤 명령어가 실행되어야하는지 정의할 때 쓰입니다.

예를 들면 이런 형태를 갖고 있습니다.

```
web: ./hello-world
```

### 2 : 애플리케이션 배포 준비 (Prepare the app)

이후 과정을 수행하려면 Heroku 콘솔 도구 [Heroku Toolbelt](https://toolbelt.heroku.com/standalone) 를 설치하고, 미리 만들어둔 Heroku 계정으로 로그인하는 과정이 필요합니다.

이 콘솔 도구는 텍스트 환경(CLI, Command-Line Interface)에서 동작하며, 배포된 애플리케이션을 구동하는 컨테이너의 성능을 조절하거나 부가 기능을 관리할 때 쓰입니다.

설치가 완료되었다면 toolbelt로 로그인합니다.

```bash
$ heroku login
```

### 3 : 컨테이너 생성 (Create the app)

[Heroku 대시보드](https://dashboard.heroku.com) 를 웹 브라우저로 열어 새로운 컨테이너를 생성합니다. 새 컨테이너를 만들 때 이름을 같이 지어주게 되는데, 이 이름을 기억해둘 필요가 있습니다.

웹 브라우저 사용이 곤란한 환경이거나 위의 방법을 선호하지 않는다면, 이전에 설치한 toolbelt를 활용하면 됩니다.

```
$ heroku create
Creating app... done, ⬢ rocky-hamlet-67506
https://rocky-hamlet-67506.herokuapp.com/ | https://git.heroku.com/rocky-hamlet-67506.git
```

이 예시에서 컨테이너 이름은 *rocky-hamlet-67506* 입니다.

### Git을 사용한 애플리케이션 배포 (Deploy using git)

직접 애플리케이션을 배포하려면 `git` 명령어를 사용하면 됩니다. 이때, 원래 있던 git 저장소에 배포용으로 사용될 원격지(remote)를 추가로 등록해야합니다.

배포할 애플리케이션(vibe.d 애플리케이션) 프로젝트가 이미 git으로 관리되고 있다는 전제하에, 아까 기억해둔 컨테이너 이름을 떠올려 새 원격지를 git 저장소 설정에 추가합니다. 여기에서는 *rocky-hamlet-67506* 를 컨테이너 이름으로 사용합니다.

```
$ heroku git:remote -a rocky-hamlet-67506
```

git 명령어를 사용해 새로운 원격지가 추가되었음을 확인할 수 있습니다.

```
$ git remote -v
heroku    https://git.heroku.com/rocky-hamlet-67506.git (fetch)
heroku    https://git.heroku.com/rocky-hamlet-67506.git (push)
```

### 빌드팩 설정 추가 (Adding the buildpack)

빌드팩(Buildpack)은 애셋(asset) 데이터를 생성하거나 소스코드를 컴파일하는, 빌드(build) 과정을 담당하는 Heroku의 도구 세트입니다. 빌드팩에 대한 자세한 설명은 [Heroku 공식 문서](https://devcenter.heroku.com/articles/buildpacks) 에 수록되어 있습니다.

vibe.d 기반 애플리케이션의 빌드의 경우 [Vibe.d 빌드팩](https://github.com/MartinNowak/heroku-buildpack-d) 을 쓸 수 있습니다.

```bash
$ heroku buildpacks:set https://github.com/MartinNowak/heroku-buildpack-d
```

기본 빌드팩 설정으로는 최신 `dmd` 컴파일러를 사용합니다만, 필요하다면 GDC나 LDC 컴파일러를 `.d-compiler` 라는 텍스트 파일을 만들고 컴파일러 정보를 명시하여 다른 컴파일러를 사용할 수 있습니다.

명시하는 방법은 단순히 컴파일러만 명시하는 방법과 버전까지 구체적으로 명시하는 두가지 방법이 있습니다.

컴파일러만 지정할 때는 `dmd`, `ldc`, `gdc` 과 같이 입력하면 되며 자동으로 최신 버전의 컴파일러가 선택됩니다.

버전까지 지정하려면 `dmd-2.0xxx`, `ldc-1.0xxx`, 또는 `gdc-4.9xxx` 처럼 구체적인 컴파일러 버전을 지정해주면 됩니다.

### 애플리케이션 배포 (Deploy the code)

이제 필요한 설정은 다 마쳤습니다. 평소에 하던 그대로, 새로운 코드를 작성하고 git으로 관리합시다.

그리고 새 버전을 배포할 때가 되었다면, 간단히 Heroku 원격지로 git push 를 하면 됩니다.

```
$ git add .
$ git commit -am "내 첫 vibe.d 애플리케이션입니다"
$ git push heroku master
Counting objects: 9, done.
Delta compression using up to 2 threads.
Compressing objects: 100% (6/6), done.
Writing objects: 100% (9/9), 997 bytes, done.
Total 9 (delta 0), reused 0 (delta 0)

-----> Fetching custom git buildpack... done
-----> D (dub package manager) app detected
-----> Building libevent
-----> Building libev
-----> Downloading DMD
-----> Downloading dub package manager
-----> Setting PATH:
-----> Initializing toolchain
-----> Building app
       Running dub build ...
Building configuration "application", build type release
Running dmd (compile)...
Compiling diet template 'index.dt' (compat)...
Linking...
       Build was successful
-----> Discovering process types
       Procfile declares types -> web
-----> Compiled slug size: 3.5MB
-----> Launching... done, v4
       https://rocky-hamlet-67506.herokuapp.com/ deployed to Heroku
To git@heroku.com:rocky-hamlet-67506.git
 * [new branch]      master -> master
```

브라우저를 열어 동작을 확인해보려면 아래와 같이 명령어를 실행합니다.

```
$ heroku open
```

### 성능 조절 (Scaling dynos containers)

배포 후에 vibe.d 애플리케이션은 Heroku의 웹 다이노(dyno) 컨테이너 안에서 동작합니다.

다이노(dyno)는 Procfile에 적힌 명령어만 실행하는 경량화된 실행 공간으로 생각하면 편합니다.

`heroku ps` 명령어로 몇 개의 다이노가 동작 중인지 체크합니다.

```
$ heroku ps
Free dyno hours quota remaining this month: 550h 0m (100%)
For more information on dyno sleeping and how to upgrade, see:
https://devcenter.heroku.com/articles/dyno-sleeping

No dynos on ⬢ rocky-hamlet-67506
```

아무 설정 없이 애플리케이션을 배포하게 되면 무료 체험용 다이노 안에 배포되며, 별도의 활성화 과정이 없으면 접속할 수 없는 상태가 됩니다.

활성화 후에도 무료 체험 플랜에서는 30분 가량 외부 접속이 없다면 일시 정지 상태가 되며, 새 접속이 들어올 경우 다시 다이노를 재가동하기에 접속이 드문 경우 약간의 응답 지연이 발생하는 특성이 있습니다.

이제 본격적으로 다이노를 가동시켜 접속할 수 있게, 아래의 명령어를 입력합니다.

```
$ heroku ps:scale web=1
```

### 서비스 로그 확인 (See the logs)

Heroku는 서비스 로그를 모두 모아 시간 순서대로 한번에 볼 수 있게 해줍니다. 명령어는 다음과 같습니다.

```
$ heroku logs --tail
```

## 더 나아가기 (More informations)

첫 vibe.d 애플리케이션을 무사히 배포했다면, 다양한 추가기능 등을 이용해 더욱 멋진 애플리케이션을 만들어보시기 바랍니다.

아래의 링크가 도움이 될 수 있습니다.

- [Postgresql](https://elements.heroku.com/addons/heroku-postgresql)
- [MongoDb](https://elements.heroku.com/addons/mongohq)
- [Logging](https://elements.heroku.com/addons#logging)
- [Caching](https://elements.heroku.com/addons#caching)
- [Error and exceptions](https://elements.heroku.com/addons#errors-exceptions)
