# repository 최신으로 동기화 

- 이 Repository 는 fork 해 온 것이므로, 원본의 Repository 의 변경 사항을 반영하려면, 다음과 같이 수행

  - remote정보 확인
  ```sh
  # git remote -v
  origin  https://github.com/yongs2/workshop-k8s-basic (fetch)
  origin  https://github.com/yongs2/workshop-k8s-basic (push)
  ```

  - 원본 Repository 를 upstream 으로 추가
  ```sh
  # git remote add upstream https://github.com/subicura/workshop-k8s-basic/
  ```

  - upstream 추가 여부 확인

  ```sh
  # git remove -v
  origin  https://github.com/yongs2/workshop-k8s-basic (fetch)
  origin  https://github.com/yongs2/workshop-k8s-basic (push)
  upstream        https://github.com/subicura/workshop-k8s-basic/ (fetch)
  upstream        https://github.com/subicura/workshop-k8s-basic/ (push)
  ```

  - upstream의 내용을 가져오기

  ```sh
  # git fetch upstream
  From https://github.com/subicura/workshop-k8s-basic
  * [new branch]      master     -> upstream/master
  ```

  - upstream 의 master branch 로 부터 Local master branch 를 merge

  ```sh
  # git checkout master

  # git merge upstream/master
  ```

  - push 명령으로 remote repository 에 반영

  ```sh
  # git push origin master
  ```
