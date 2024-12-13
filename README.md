### 개요

본 깃허브 블로그는 [so-simple-theme](https://github.com/mmistakes/so-simple-theme) 테마를 활용하여 제작하였습니다.
더아이엠씨 AI 블로그 접속 주소는 [aitheimc.github.io](aitheimc.github.io)입니다.

### 깃허브 블로그를 만들기 위해 참고한 사이트

- 깃허브 레퍼지토리 생성 및 Github Desktop와 VSCode 설치
> https://wlqmffl0102.github.io/posts/Making-Git-blogs-for-beginners-1/

- 백엔드 코드 ruby와 정적 사이트 생성기 jekyll 관련 내용
> https://wlqmffl0102.github.io/posts/Making-Git-blogs-for-beginners-2/

로컬에 ruby 및 jekyll을 설치하고 본 레퍼지토리를 `clone` 하여 `jekyll serve` 명령어를 실행하면 로컬에서 확인 후 깃허브에 `push` 할 수 있습니다. 

- so-simple-theme 사용법은 아래 블로그를 통해 확인할 수 있습니다.
> https://olvimama.github.io/blog/gitpages-about-so-simple-theme

> https://github.com/mmistakes/so-simple-theme

> https://github.com/olvimama/olvimama.github.io/tree/master

### 문서 참고 없이 커스텀한 내용

#### 폰트

`_sass /so-simple /_variables.scss`  파일 내용을 아래 와같이 수정 하였으며, [구글 폰트](https://fonts.google.com/specimen/Nanum+Gothic?lang=ko_Kore)를 활용함.

```css
$nanum-gothic-font-family: "Nanum Gothic", sans-serif !default; /* 폰트 패밀리에 나눔고딕체 추가 */
$title-font-family: $nanum-gothic-font-family !default; /* title-font-family 를 nanum-gothic-font-family로 변경*/
```

#### author

`_includes / page-author.html` 파일에 아래 코드 추가 하여 position, description 레이아웃 추가함

```html
{%- if author.position -%}
<div class="author-name">
    <span class="p-name">{{ author.position }}</span>
</div>
{%- endif -%}
{%- if author.description -%}
<div class="author-name">
    <span class="p-name">{{ author.description }}**</span>
</div>
{%- endif -%}
```
추가적으로 `name` 부분에 볼드체 추가

```html
 {% if site.data.text[site.locale].by %}<em>{{ site.data.text[site.locale].by }}</em> {% endif %}<span class="p-name"><b>{{ author.name }}</b></span>
```

#### GA(google analysis) 코드 삽입

`_layouts / page.html` 내 header 사이에 구글 script 문구를 삽입하고, _config.yml 파일에 UA 입력


⚠️ **Warning:** 블로그 글을 작성하다가, IP, 비밀번호 등 중요한 정보가 어느 브랜치든 실수로 업로드 되면 레포지토리를 폭파시키고 다시 만들어야 하니 반드시 공유해주세요. 리포지토리를 폭파시키고 다시 만들면 `collaborator` 초대 작업만 다시 하면 됩니다.