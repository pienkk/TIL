>Github Copilot은 MicroSoft와 ChatGPT로 유명한 OpenAI의 합작으로 2021년 06월 발표한 소스코드 자동완성 인공지능이다.
>인공지능 모델은 GPT-3모델의 코드 생성 특화버전 Codex 모델이 사용되었다.

## 인공지능과 프로그래밍

ChatGPT가 발표된 이후 코드를 작성함에 있어 코드를 작성해 주거나, 코드리뷰를 해주거나 하는 등의 방식으로 많은사람들이 인공지능을 프로그래밍에 활용하고 있다.
Github도 3월 22일 GPT-4를 기반으로 한 Copilot X를 발표하는 등 앞으로의 개발환경에 인공지능의 활용은 무궁무진하게 커질것이다.

## Copliot

### 사용가능 IDE

Copilot을 일반사용자도 사용할수 있게 된건 2022년 6월에 정식 런칭이후 부터 사용이 가능해졌다.
Copilot을 사용하려면 IDE에 확장프로그램을 설치해 사용해야 하는데, 확장프로그램이 존재하지 않는 IDE에는 Copliot을 사용할 수 없다.
Copliot을 지원하는 IDE는 Visual Studio, Visual Studio Code, JetBrains IDE, Neovim이 있다. 현재 가장인기많은 Visual Studio Code부터 유료지만 강력한 기능덕에 인기가 많은 JetBrain의 IDE 그리고 특이하게 Neovim도 지원을 해 Copliot을 사용하기 위해 새로운 IDE에 적응할 필요는 없을것 이다.

### 주요 기능

![](https://velog.velcdn.com/images/kisuk623/post/2d88b2f6-6429-4723-a5c9-ff4cc26e3da4/image.png)


위 사진은 Copliot 공식홈페이지에서 가져왔다. 파란색으로 드래그된것이 Copilot이 작성해준 코드이다. 분산형 차트를 그리는 함수명을 만들면 Copilot이 해당 함수명을 읽어 함수의 내용을 제시해 준다.
제시된 내용이 마음에 들지 않을 경우 다른 내용을 받아볼 수도 있고, 그 중에 하나를 수락하면 해당 코드는 자동으로 작성된다.
이외에도 코드만 작성해주는것이 아닌 해당 함수의 주석을 생성해 주기도 하고, 단순 반복의 코드를 줄이게 해준다.


### Copilot 설치하기

Copilot은 현재 60일간 무료체험을 해볼수 있으나, 60일의 무료체험이 종료되면 매달 10달러의 요금을 지불해야 사용할수 있다. 

![](https://velog.velcdn.com/images/kisuk623/post/fc2737c8-b8b6-4d89-9736-d74e2fee0eaf/image.png)

Copilot을 사용하기 위해선 [Github Copilot](https://github.com/features/copilot) 링크에 들어가 Start my free trial아이콘을 클릭해 무료체험 시작 버튼을 누른다.

![](https://velog.velcdn.com/images/kisuk623/post/39fb44e7-5bd8-4d6c-9ae4-feaf0a542de1/image.png)


그 후 무료체험 후 이용할 플랜을 고른다. 해당 플랜은 언제든지 변경이 가능하며, 어느것을 고르든 과금이 되지는 않는다. 우선 1달짜리 플랜으로 선택하자.
선택후 Get access to GitHub Copilot버튼을 누르면, 카드정보를 입력하는 페이지가 나오게 된다. 해당 페이지에서 카드정보를 입력하면 해당 카드가 유효한지 확인하는 1달러가 결제되고 곧이어 환불이 될것이다.

![](https://velog.velcdn.com/images/kisuk623/post/f955c106-55f4-4e3d-88d1-67d24f358b81/image.png)


그 후 해당화면이 나오게 되는데 코드 제안목록에 다른사람이 작성한 공개코드를 제안할 것인가의 여부와 나의 코드를 제품 개선에 할용하는데 동의할것인지 여부를 묻게 된다. 이건 사용자에 따라 다르게 설정하게 될것이다.

가입에 성공했다면, 사용할 IDE에서 Copilot 확장프로그램을 설치해야 한다. 설명은 현재 가장 인기많은 Visual Studio Code에서 설명하겠다.
Visual Studio Code 마켓플레이스에서 Copilot을 찾아 설치를 하면 현재 IDE에 Github계정으로 로그인이 되어있지 않은 상태라면 로그인 요청 팝업이 뜨게 될것이다. 해당 팝업을 클릭해 Github로그인을 진행하면 Copilot이 정상적으로 작동하게 된다.
![](https://velog.velcdn.com/images/kisuk623/post/87af94f6-1685-49af-8f42-c4087f1ecea8/image.png)


### 직접 사용해보자

![](https://velog.velcdn.com/images/kisuk623/post/791cf6fe-7235-4677-bbbb-e57217eb8d6a/image.mov)


Dto클래스를 Entity클래스형식으로 변환하기 위해 toEntity 메소드를 선언했더니 Copilot이 메소드를 읽고 해당 내용을 작성해 줬다.
나는 해당 기능을 사용하면서 매우 큰 생산성 향상을 느끼게 되었다. 코드를 작성하며, 코드의 작성 로직을 고민하는데에도 시간이 많이 소요가 되지만, 단순 반복코드를 작성하는데에도 시간을 많이 소요하게 되는데, Copilot이 단순 코드를 작성해줘서 해당 시간을 크게 절약하게 되었다.

![](https://velog.velcdn.com/images/kisuk623/post/281a25bd-26e3-46ae-9550-c249b0f85b1e/image.mov)


이외에도 테스트코드를 위한 seed를 만들때에도 코드의 문맥을 읽어 데이터를 만들어줘 편하게 작업을 진행했다.

## 마치며

처음 Copilot과 ChatGPT를 비교해 봤을 때, ChatGPT가 인공지능 모델이 더 최신화되고 높기 때문에 단순히 ChatGPT가 좋다고 생각했었는데, 직접 사용해본 결과 Copilot과 ChatGPT는 조금 다른 느낌이었다. ChatGPT는 내가 모르는게 있을때 물어보는 구글같은 느낌이고, Copilot은 항상 나의 곁에 있는 나의 분신 같은 느낌이었다. Copilot이 내가 아무런 생각도 하지않고 코드를 작성할수 있게 해주진 않는다. 하지만, 단순 반복이나 간단한 코드의 생산성을 높여줘, 남는 시간에 코드를 수정하는 등 고품질의 코드를 생각하고 작성할수 있었다.
시간이 갈수록 인공지능이 고도화 되고있는데, 시대에 뒤쳐지지 않기 위해서는 인공지능을 활용해 더욱 좋은 코드를 작성해야 할것이다.

