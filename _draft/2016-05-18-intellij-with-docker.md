# Media-Site with Docker

## Docker 설치 

다운로드:
* [https://www.docker.com/products/docker-toolbox](https://www.docker.com/products/docker-toolbox)

참조: 
* [Docker 설치](https://docs.docker.com/engine/installation/mac/#installation)

### Docker-machine 
참조: 
* [what is docker machine](https://docs.docker.com/machine/overview/#what-is-docker-machine)

맥에 설치된 Docker 용 Virtual Host 실행 및 정상 설치 확인.
참조: 
* [From your shell](https://docs.docker.com/engine/installation/mac/#from-your-shell)
``` bash 
$ docker-machine start
$ docker-machine ls
$ docker-machine env default
$ eval "$(docker-machine env default)"
$ docker run hello-world
```
결과:
![docker_run_success](/Users/min/Desktop/docker_run_success.png)

## Plugin Docker integration 설치

> Intellij > Preferences... > Plugins > Install JetBrains plugin... > Search Docker integration > Install > Restart Intellij IDEA

![Intellij Docker Integration](/Users/min/Desktop/docker_integration_plugins.png)
## Docker configuration 생성

> Intellij > Preferences... > Build, Execution, Deployment > Clouds > + 버튼 클릭 > Docker 선택

![Clouds > Docker config ](/Users/min/Desktop/plugins_docker_config.png)

## Docker Deployment run configuration

> Run > Edit Configurations > + 버튼  클릭 > Docker Deployment

참조: 
* [ Docker Deployment Configuration ](https://www.jetbrains.com/help/idea/2016.1/clouds.html#docker)
* [ Run/Debug Configuration: Docker Deployment](https://www.jetbrains.com/help/idea/2016.1/run-debug-configuration-docker-deployment.html?origin=old_help)
* [Docker Support in IntelliJ IDEA 14.1](https://blog.jetbrains.com/idea/2015/03/docker-support-in-intellij-idea-14-1/)

