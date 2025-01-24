---
title: "Github 대용량 파일 업로드 삽질...."
layout: single
Typora-root-url: ../
categories: TroubleShooting
tag: Git
use_math: true
---

![]({{site.url}}/images/2024-08-05-gihub-largefile/Stella.png)
### GIF는 어지간하면 올리지 말자...
회고록을 올리면서 영상 데이터는 올릴 수가 없어서 MOV 파일을 Gif로 변환하여 깃허브에 push를 했다.

```shell
Total 20 (delta 3), reused 0 (delta 0), pack-reused 0
remote: Resolving deltas: 100% (3/3), completed with 3 local objects.
remote: error: Trace: 4ab160377c35c233a1d674a88230e5576ee58feb14500e0ac808193124a5dde9
remote: error: See https://gh.io/lfs for more information.
remote: error: File content/memoir-sesac-hackathon/final.gif is 191.01 MB; this exceeds GitHub's file size limit of 100.00 MB
remote: error: GH001: Large files detected. You may want to try Git Large File Storage - https://git-lfs.github.com.
To https://github.com/Dongckim/dongckim.github.io.git
 ! [remote rejected] master -> master (pre-receive hook declined)
error: failed to push some refs to 'https://github.com/Dongckim/dongckim.github.io.git'
```

이게 도대체 뭔데...자세히 보니, gif의 용량이 너무 커서 원격서버에 올릴 수 없다는 경고문구였다<br/>
오케이 그럼 GIF 용량을 줄여야겠다~~ 너무 단순히 생각했었다. 그냥 gif 픽셀값을 줄여서 업로드 했다.

```shell
Total 25 (delta 6), reused 0 (delta 0), pack-reused 0
remote: Resolving deltas: 100% (6/6), completed with 3 local objects.
remote: warning: File content/memoir-sesac-hackathon/final.gif is 58.12 MB; this is larger than GitHub's recommended maximum file size of 50.00 MB
remote: error: Trace: b94cfc92d455b215a00a0974b0b03c89690cf64a990b7e499a12110b4cf7c870
remote: error: See https://gh.io/lfs for more information.
remote: error: File content/memoir-sesac-hackathon/final.gif is 191.01 MB; this exceeds GitHub's file size limit of 100.00 MB
remote: error: GH001: Large files detected. You may want to try Git Large File Storage - https://git-lfs.github.com.
To https://github.com/Dongckim/dongckim.github.io.git
 ! [remote rejected] master -> master (pre-receive hook declined)
error: failed to push some refs to 'https://github.com/Dongckim/dongckim.github.io.git'
(base) dongchankim@dongchankimui-MacBookAir dongckim.github.io % 
```
네???

기존의 파일은 삭제되지 않을 뿐더러, 새로운 파일 조차도 깃허브 제한에 걸리고 있는 모습이었다.

Sibal 구글링해보자.. 제일 먼저 깃허브 공식문서를 봤다.
![]({{site.url}}/images/2024-08-05-gihub-largefile/size.png)

아...공식문서에서 조차 1GB 미만을 강력히 권고하고 있다.. 코드와 다양한 사진들을 함께 푸시했기 때문에 전체 용량으로 봤을 때 위험한 상황임으로 판단 되었다.

오케이 그럼 삭제했는데 왜 삭제가 안되었다고 하지?

![]({{site.url}}/images/2024-08-05-gihub-largefile/history.png)

로컬에서는 삭제되었더라도, 히스토리나 캐시데이터가 남아있을 수 있다는 걸 알았다. 해당 커밋기록이나, 캐시들을 전부 지워줘야 할 꺼 같았다.

```shell
brew install git-lfs

git add .gitattributes
git commit -m "Track large GIF file with Git LFS"

git add content/memoir-sesac-hackathon/final.gif
git commit -m "Add GIF file with LFS"
```
GPT에서 추천하는 방식이었다. LFS라는 원격 Git Large File Storage로, 해당 파일을 그대로 올릴 수 있도록 도움을 줄 수 있는 것처럼 보여졌다.

결론적으로는 아무 변화가 없었다... (역시 GPT는 믿고 걸러야 한다)

```shell
git rm --cached content/memoir-sesac-hackathon/final.gif
git commit -m "Remove large GIF file"
```
해당 커맨드를 이용해서 로컬에 남아있는 히스토리를 지울 수 있지만, 캐시데이터는 여전히 남아있는 것을 보여졌다.

```shell
git filter-branch --index-filter "git rm -rf --cached --ignore-unmatch bringup" HEAD
git push origin --force --all
```
위의 `git rm` 명령어를 사용할 경우 HEAD쪽에서는 삭제되어진 것으로 보여졌지만, 사실 History에는 여전히 남아있다고 한다.
따라서 다시 되살릴 수 있는 여지가 있기 때문에 repo자체의 사이즈가 줄지 않는다고 한다. 완전삭제의 느낌과는 거리가 있어보인다.

해당 명렁어를 통해 git 전체를 돌면서 관련 파일을 모두 삭제할 수 있었다. 뭔가 진행되어보여서 기분 좋게 기다렸다...

`push`도 안되던 것이 `--force`를 통해 강제 입력이 가능했고, 나름 희망적이었던 것 같다.
근데 여전히...배포가 중단되는 것이다!!!! 으ㅏㅏ아아ㅏ아아아아ㅏㅏ앙!!

한 2시간 여 삽질을 하다가 도저히 안되겠어서 수동삭제의 방식을 택하게 되었다...
```shell
find . -type f -size +100M
```
일단 100M 넘는 파일 다 찾아.

오? 문제가 되었던 final.gif 파일이 보여졌다. 경로를 바탕으로 뒤지기 시작했다. 처음으로 node_modules파일을 열어보는 계기가 된거 같기도 하다 ㅋㅋㅋㅋㅋㅋㅋ

아니 여기에 gif파일이 몰래 숨어있지 아니한가!!!!!! 바로 지워버려!!

떨리는 마음으로 배포 시작...
![]({{site.url}}/images/2024-08-05-gihub-largefile/published.png)

Published!!!!! 배포완료!!!

![]({{site.url}}/images/2024-08-05-gihub-largefile/die.png){: .img-width-half}

종료시각 새벽 3시 34분...