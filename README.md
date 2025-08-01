## 개요

본 깃허브 블로그는 [so-simple-theme](https://github.com/mmistakes/so-simple-theme) 테마를 활용하여 제작하였습니다.
더아이엠씨 AI 블로그 접속 주소는 [aitheimc.github.io](https://aitheimc.github.io/)입니다.

## 더아이엠씨 기술 깃허브 블로그를 만들기 위해 참고한 사이트

- 깃허브 레퍼지토리 생성 및 Github Desktop와 VSCode 설치 관련 내용
> https://wlqmffl0102.github.io/posts/Making-Git-blogs-for-beginners-1/

- 백엔드 코드 ruby와 정적 사이트 생성기 jekyll 관련 내용
> https://wlqmffl0102.github.io/posts/Making-Git-blogs-for-beginners-2/

- so simple 테마 환경 설정 및 자주 발생하는 문제 관련 내용
> https://olvimama.github.io/blog/gitpages-about-so-simple-theme


## 문서 참고 없이 커스텀한 내용

### 폰트

#### 일반 사항
전반적으로 폰트 크기를 다운 사이즈 하기 위해 `_sass /so-simple /_variables.scss` 파일을 아래와 같이 수정
```css
/* Fluid type */
$base-font-size: 14px !default;
//$base-font-size: 16px !default;
$min-vw: $small !default;
$max-vw: $xlarge !default;
$min-font-size: 14px !default;
$max-font-size: 16px !default;
//$min-font-size: 16px !default;
//$max-font-size: 18px !default;
``` 

한글 폰트 수정을 위해 `_sass /so-simple /_variables.scss`  파일을 수정 하였으며, [구글 폰트](https://fonts.google.com/specimen/Nanum+Gothic?lang=ko_Kore)를 활용함.

```css
$nanum-gothic-font-family: "Nanum Gothic", sans-serif !default; /* 폰트 패밀리에 나눔고딕체 추가 */
$IBM-Plex-sans-KR-font-family: "IBM Plex Sans KR", sans-serif !default; /* 폰트 패밀리에 IBM Plex 추가 */
$serif-font-family: "Lora", serif !default;
$sans-serif-font-family: "Source Sans Pro", sans-serif !default;
$monospace-font-family: Menlo, Consolas, Monaco, "Courier New", Courier, monospace !default;

$base-font-family: $IBM-Plex-sans-KR-font-family !default; /* base-font-family: 를 $IBM-Plex-sans-KR-font-family로 변경*/
$headline-font-family: $IBM-Plex-sans-KR-font-family !default; /* headline-font-family 를 $IBM-Plex-sans-KR-font-family로 변경*/
$title-font-family: $IBM-Plex-sans-KR-font-family !default; /* title-font-family 를 $IBM-Plex-sans-KR-font-family로 변경*/
$description-font-family: $IBM-Plex-sans-KR-font-family !default; /*description-font-family 를 $IBM-Plex-sans-KR-font-family로 변경*/
$meta-font-family: $IBM-Plex-sans-KR-font-family !default; /* meta-font-family 를 $IBM-Plex-sans-KR-font-family로 변경*/
//$base-font-family: $sans-serif-font-family !default; 
//$headline-font-family: $sans-serif-font-family !default; 
//$title-font-family: $serif-font-family !default;
//$description-font-family: $serif-font-family !default;
//$meta-font-family: $serif-font-family !default;
```

#### 글 목록의 제목 폰트 수정

목록에서 제목 크기조절 및 이탤릭체 제거를 위해 `_sass /so-simple /_entries.scss` 파일을 아래와 같이 수정함.

```css
.entry-title {
  margin-bottom: 0.5rem;
  font-family: $title-font-family;
  font-weight: $entry-title-weight;
  /* 폰트 사이즈 강제 추가 반응형 안됨*/
  font-size: 18px !important;
  /*font-style: italic;*/
  letter-spacing: -1px;
  word-wrap: break-word; /* break long words that could overflow */

...중략...
```

블로그 내 제목에서 크기조절 및 이탤릭체 제거를 위해 `_sass /so-simple /_page.scss` 파일을 수정함.

```css
.page-title {
  @include fluid-type($min-vw, $max-vw, 30px, 40px); /* 픽셀 크기 30, 40로 변경*/
  margin-bottom: 0.5em;
  font-family: $title-font-family;
  font-weight: $page-title-weight;
  # font-style: italic;
  letter-spacing: -2px;
}
```
#### 코드쉘 내 글자 크기 축소 
`_sass/so-simple/_reset.scss` 내용 수정
```css
code,
kbd,
samp {
  font-family: monospace, monospace; /* 1 */
  font-size: 0.80rem; /* 2 */
}
```

### author

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

### 테이블 스타일
`_sass /so-simple /_tables.scss` 수정

```css
/* ==========================================================================
   Simple Table (compact rows + inner borders only)
   ========================================================================== */

table {
  width: auto;
  max-width: 100%;
  border-collapse: collapse;
  font-size: 0.95rem;
  word-break: keep-all;
  /* 바깥쪽 테두리 제거 */
  border: none;
}

th,
td {
  padding: 0.4em 0.75em;
  border-top: 1px solid #ccc;
  border-bottom: 1px solid #ccc;
  /* 좌우 구분선 제거 */
  border-left: none;
  border-right: none;
}

th {
  font-weight: 600;
  text-align: left;
  background-color: #f0f0f0;
  color: #222;
}

tr {
  vertical-align: middle;
}

thead tr {
  border-bottom: 2px solid #999;
}

tbody tr:hover {
  background-color: #f5f5f5;
}
``` 

### 하이라이트 스타일 수정
`_sass /so-simple /_syntax-highlighting.scss` 에 `.highlight pre` 부분 수정

```css
.highlight pre {
  width: 100%;
  color:white;
  background-color: rgb(40, 40, 40);
}
```

`_sass /so-simple /_skin.scss`에 폰트 수정
수정 전 색상은 주석처리해두었으며, 내용이 길어 본 README 파일에 기록하지 않음


### 인용문 스타일 수정
`_sass /so-simple /_base.scss` 에 `blockquote` 부분 수정
```css
blockquote,
q {
  margin: 0.25em 0;
  padding-left: 0.75em;
  border-left: 2px solid #ccc;
  border-top: none;
  border-bottom: none;
  border-right: none;
  background: none;
  color: #333;
  font-style: normal; /* 이탤릭 제거 */
}

footer {
  font-size: 0.75em;
  font-style: normal;
  margin-top: 0.25em;
}

footer::before {
  content: "— ";
  padding-right: 0.25rem;
}

code,
kbd,
samp,
pre {
  font-family: monospace;
}

kbd {
  display: inline-block;
  margin: 0 0.1em;
  padding: 0.05em 0.35em;
  color: #000;
  border: 1px solid #ccc;
  border-radius: 2px;
  background-color: #f5f5f5;
  font-size: 0.85em;
  text-shadow: none;
  box-shadow: none;
}
```

### 검색 최적화 : GA(google analysis) / 네이버 코드 삽입

`_includes / head.html` 내 header 사이에 구글 script 문구를 삽입하였으며, 변경 내용은 아래와 같습니다.
```html
  <!-- DMC 팀 요청으로 title 및 description 테그 추가-->
  <title>더아이엠씨 테크 블로그</title>
  <meta name="description" content="누구나 접근하기 쉬운 AI를 만들기 위한 더아이엠씨 팀 이야기">
  <!-- Google tag (gtag.js) -->
  <script async src="https://www.googletagmanager.com/gtag/js?id=G-WX1J8PL1JX"></script>
  <script>
    window.dataLayer = window.dataLayer || [];
    function gtag(){dataLayer.push(arguments);}
    gtag('js', new Date());

    gtag('config', 'G-WX1J8PL1JX');
  </script>
  <!-- Google tag (gtag.js) 
  <script async src="https://www.googletagmanager.com/gtag/js?id=G-PRCBSBTR5H"></script>
  <script>
    window.dataLayer = window.dataLayer || [];
    function gtag(){dataLayer.push(arguments);}
    gtag('js', new Date());

    gtag('config', 'G-PRCBSBTR5H');
  </script>-->
  <!-- Google Tag Manager -->
  <script>(function(w,d,s,l,i){w[l]=w[l]||[];w[l].push({'gtm.start':
    new Date().getTime(),event:'gtm.js'});var f=d.getElementsByTagName(s)[0],
    j=d.createElement(s),dl=l!='dataLayer'?'&l='+l:'';j.async=true;j.src=
    'https://www.googletagmanager.com/gtm.js?id='+i+dl;f.parentNode.insertBefore(j,f);
    })(window,document,'script','dataLayer','GTM-KH3JR7SV');</script>
  <!-- End Google Tag Manager -->
  <!-- DMC 팀 요청으로 meta 테그 추가-->
  <meta name="naver-site-verification" content="00f64abc1586c4491dd51f8628fe9ab18fca0c40" />
  <meta property="og:type" content="website"> 
  <meta property="og:title" content="The IMC 테크 블로그">
  <meta property="og:description" content="누구나 접근하기 쉬운 AI를 만들기 위한 The IMC의 AI 모델링팀 이야기">
  <!-- END DMC 팀 요청으로 meta 테그 추가-->
  <!-- 구글 서치 컨솔 verification-->
  <meta name="google-site-verification" content="dbLji7nR7NCquXGY4WxvIyTy9Uib8bSsrdHQrLv_V8g" />
```

_config.yml 파일에 UA 입력

구글 seo scipt는 header와 body 맨끝에 추가해둠.

[주의] 마이너한 교체는 기입이 안되어 있을 수 있습니다.

## 블로그 작성 전 읽어야 하는 문서 목록

- [더아이엠씨 AI 블로그 post layer 작성법](https://aitheimc.github.io/guide-for-blog-write/)
- [AI 블로그 작성을 위한 기본적인 깃허브 정보](https://aitheimc.github.io/basic-info-git-and-github/)

**[참고]** 지향하는 블로그 디자인을 참고 하고 싶다면 아래 사이트를 참고하세요!
> https://aws.amazon.com/ko/blogs/tech/

⚠️ **Warning:** 블로그 글을 작성하다가, IP, 비밀번호 등 중요한 정보가 어느 브랜치든 실수로 업로드 되면 레포지토리를 폭파시키고 다시 만들어야 하니 반드시 공유해주세요. 리포지토리를 폭파시키고 다시 만들면 `collaborator` 초대 작업만 다시 하면 됩니다.