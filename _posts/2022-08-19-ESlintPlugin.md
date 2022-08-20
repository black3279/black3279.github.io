---
layout: post
author: Bruce Lee
title: Better JavaScript with ESlint, Airbnb, Husky & Lint staged
---

## 👨‍🎓 1. ESlint 란<br/><br/>
- JavaScript용 도구로 ESLint는 ES 와 Lint를 합친 것이다.<br/>
  ES는 Ecma Script로서, Ecma라는 기구에서 만든 Script, 즉, 표준 Javascript를 의미한다.<br/>
  Lint는 에러가 있는 코드에 표시를 달아놓는 것을 의미한다.<br/>
  따라서, ESLint는 자바스크립트 문법에서 에러를 표시해주는 도구이다!<br/>
  우리는 에러로서, 정말 문제가 되는 부분만을 지정할 수도 있고, 아니면 전반적인 코딩스타일(ex. tab 설정, ; 여부 등)까지 지정할 수도 있다.<br/>
  많은 사람들과 협업할때 특히 유용한다. 에러와 코딩 스타일을 잡아주기 때문에 한 사람이 코딩한 것처럼 된다! 😀<br/>
- 코드의 가독성을 높혀주고 에러나 컨벤션에 관한 경고 해주는 유명한 툴이 있는데<br/>
  바로 ESLint와 Prettier이다. 형식과 규칙에 어긋나는 부분들을, format에 대한 경고와 문법오류에 대한 경고들을 linterror로 표시해주는 기능을 제공하는 extension이다.<br/>
- 코드를 분석해 문법적인 오류나 안티 패턴을 찾아주고 일관된 코드 스타일을 유지(포맷팅)하여 개발자가 쉽게 읽도록 코드를 만들어준다.<br/>
  <br/><br/>
- 왜 사용해야 하는가 ?<br/>
    - 우리 모두가 알고 있듯이 Javascript는 동적 언어이다. 즉, 오류를 만들고 잘못된 코드를 작성할 수 있는 여지가 많습니다. 따라서 이러한 오류 중 일부를 방지하기 위해 ESLint가 있다.<br/>

### 2. Airbnb Style <br/><br/>
- ESLint는 스타일 가이드를 좀 더 편리하게 적용하기 위해 사용하기도 한다. 많이 쓰이는 것은 Airbnb Style Guide, Google Style Guide 등이 있다.<br/>
- 에어비앤비 스타일 가이드는 무엇일까? 에어비앤비 스타일 가이드는 좋은 코드를 작성하기 위한 가이드라인과 몇 가지 일반적인 모범 사례이다. Airbnb 스타일 가이드는 Github에서 가장 주목받는 스타일 가이드 중 하나이다. 참고: eslint-config-airbnb-base에는 React 에 대한 린트 규칙이 제공되지 않습니다. React 에 대한 규칙을 원하는 경우 eslint-config-airbnb 사용을 고려한다.<br/>
- Airbnb ESLint에는 eslint-config-airbnb와 eslint-config-airbnb-base가 있는데 base는 리액트 관련 규칙을 포함하지 않는 것이다. base로 설치하려면 아래 명령어에서 airbnb 뒤에 -base 를 붙인다.<br/>
  <br/>
```javascript
module.exports = {
  env: {
    browser: true,
    es2021: true,
  },
  extends: ["airbnb-base", "prettier"],
  parserOptions: {
    ecmaVersion: 12,
    sourceType: "module",
  },
  plugins: ["prettier"],
  rules: {
    "prettier/prettier": "error",
  },
};
```
<br/>

- git Repo : https://github.com/airbnb/javascript / https://github.com/ParkSB/javascript-style-guide<br/>
- eslint-airbnb-base : https://www.npmjs.com/package/eslint-config-airbnb-base<br/>
- eslint-plugin-security : https://www.npmjs.com/package/eslint-plugin-security<br/>
- 예시 ) timing Attack은 Side Channel Attack의 한 종류이다. Side Channel Attack은 간단히 말하면 암호 체계 파악을 위해 암호 알고리즘을 분석하는 게 아닌 암호화에 소요된 시간, 전력, 전자기파, 소리 등을 분석하여 정보를 얻는 공격 방법이다.<br/>
<br/>
일반적으로 고객의 계정 비밀번호를 DB에 저장할 때 암호화 알고리즘을 이용해 암호화한 뒤 저장하게 되는데, 이 때 걸리는 시간을 분석하여 원본을 추적하는 것을 Timing Attack이라고 한다.<br/>
<br/>

### 3. husky, lint-staged 의 Git hook 을 이용한 ESlint 자동화 <br/><br/>
#### 1) husky? lint-stage?<br/>
>   우리는 ESLint를 프로젝트에 적용시킬 때는 협업하는 모든 사람들이 같은 규칙 내에서 코딩을 하는 것을 예상한다. 하지만 가끔은 규칙을 지키지 않고 깃헙에 코드를 푸시할 때가 생긴다.<br/>
많은 사람들의 경우에도 가끔 오랜 코딩에 지쳐서 깜빡하고 ESLint 확인을 안하고 푸시할 때가 있었다.<br/>
그래서 우리는 git commit 또는 git push와 같은 git 이벤트가 일어나기 전에 우리가 원하는 스크립트를 실행하기 위해서 git 이벤트 사이에 갈고리(hook)를 걸어주는 것이다.<br/>
이것을 git hook 제어라고 한다. 우리는 이런 git hook 제어를 위해서 husky 라이브러리를 사용할 것이다.<br/>
그러면 lint-staged는 뭐냐? 우선 stage 상태를 이해해야한다. 파일들이 git add로 커밋 대상이 된 상태를 stage 상태라고 한다.<br/>
stage 상태의 git 파일에 대해 lint와 우리가 설정해둔 명령어를 실행해주는 라이브러리다.<br/>

<br/>
- 프로그래밍에서 hook이란 특정 이벤트 또는 함수가 호출되기 전/후에 호출이 되는 코드를 의미한다.<br/>
- Git은 특정 상황에 특정 스크립트를 실행할 수 있도록 하는 Hook이라는 기능을 지원하고 있다. 별도로 설치할 것은 없고 모든 git repository에서 지원한다. git으로 관리하고 있는 폴더에서 cd .git/hooks/ 명령어를 치고 ls 명령어를 쳐보면 .sample 확장자로 되어있는 파일이 13개가 있다. 이는 git hook이 지원하는 특정 상황이 13개인 것을 알 수 있다. 해당 hook을 사용하려면 .sample 확장자만 지우면 된다.<br/>
- 이번에 사용해볼 git hook은 pre-commit이다.<br/>
- git hooks는 .git 디렉터리에 저장되기 때문에 git repository에 원격으로 저장하고 관리할 수 없다. 따라서 git hook을 공유하기 위한 방법에는 여러 가지가 있지만 그중 husky를 사용해보고자 한다.<br/>
- husky란 Node.js 개발 환경에서 git hook을 편리하게 사용할 수 있게 만들어주는 도구이다. husky를 사용하면 프로젝트 별로 commit, push 등과 관련된 정책을 관리하고 공유할 수 있다.<br/>
- git hook을 활용해서 commit 하기 전에 lint를 실행해서 코드를 검사할 수 있게 됐다. 하지만 검사해야하는 확장자에 해당하는 모든 파일을 검사하려면 굉장히 오랜 시간이 소요될 것이다. 이때 lint-staged를 함께 사용하면 변경된 파일만 검사할 수 있다. 즉 git staging area에 올라온 파일만 검사할 수 있다<br/>
- git repo : https://github.com/okonet/lint-staged / https://github.com/typicode/husky<br/>
<br/>

# 정리<br/>
# Lint-Staged<br/>
<br/>
공식 repo에 가보면 한줄소개로 어떤 역할을 하는 라이브러리로 볼 수 있다.<br/>
<br/>
Run linters against staged git files and don't let 💩 slip into your code base!<br/>
<br/>
lint를 staged에 있는 파일을 체크함으로서, lint:fix를 통과하지 않는 파일은 커밋 하지 않게 도와준다.<br/>
<br/>
설치<br/>
<br/>
npm install --save-dev lint-staged@next<br/>
<br/>
# Husky<br/>
githooks 를 npm 을 통해서 관리할 수 있게 도와주는 라이브러리 이다.<br/>
<br/>
대응되는 npm script를 pacakge.json에서 매칭시켜서 쉽게 적용할 수 있게 도와준다.<br/>
<br/>
# Lint-Staged & Husky 적용<br/>
husky는 gut hooks를 통해서 commit이나 push 전에 행동을 도와준다.<br/>
<br/>
npm install husky --save-dev<br/>
// package.json<br/>
```javascript
"husky": {
    "hooks": {
        "pre-commit": "npm test",
        "pre-push": "npm test",
        "...": "..."
    }
}
```
<br/>
그러면 이 두가지를 앞서 설정한 부분과 함께 적용해보겠다.<br/>
각 npm 모듈들을 설치 후 npm install --save-dev lint-staged@next husky<br/>
<br/>
packae.json에 husky 설정과 lint-staged 설정을 다음과 같이 추가해준다.<br/>
<pre>
<code>
    "husky": {
        "hooks": {
            "pre-commit": "lint-staged"
        }
    },
    "lint-staged": {
        "*.{js,vue}": [
            "eslint --fix",
            "prettier --write",
            "git add"
        ]
    }
</code>
</pre>
commit 을 했을 때<br/>
<br/>
[eslint—fix, prettier—write]를 문제 없이 통과해야 commit을 할 수 있게 된다.<br/>
<br/>
# 다시 정리하면<br/>
## staging에 있는 파일을 npm 으로 다루기 위해서 → lint-staged<br/>
## git hooks를 npm으로 다루기 위해서 → husky 를 이용한다.<br/>
## 이 두가지를 조합해서 commit 하기 전에 파일들을 lint 룰이 적용된 파일만 커밋할 수 있게 도와준다.<br/>