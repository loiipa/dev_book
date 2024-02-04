# Hugo with github 세팅
## Window powershell 기준

### 1. install go (programming language)
https://go.dev/doc/install

### 2. Hugo Clone
https://github.com/gohugoio/hugo

주의 - Theme에 따라 Extension clone을 해야할 수 있어서 적용하고 싶은 theme에서 extension 사용 유무를 먼저 확인

```bash
# extension
CGO_ENABLED=1 go install -tags extended github.com/gohugoio/hugo@latest

# extension에서 해당 명령이 수행되지 않는 경우
set CGO_ENABLED=1
go install -tags extended github.com/gohugoio/hugo@latest
```

### 3. Theme Clone
https://themes.gohugo.io/

여기서 원하는 테마를 선택 후 아래의 절차 수행

#### * Hugo Book 기준
Hugo book은 submodule을 이용하라는 가이드를 제시해준다.
```bash
hugo new site mydocs; cd mydocs
git init
git submodule add https://github.com/alex-shpak/hugo-book themes/hugo-book
cp -R themes/hugo-book/exampleSite/content.en/* ./content

# 서버 실행 http://localhost:1313/
hugo server --minify --theme hugo-book
```

### 4. GitHub Host
https://gohugo.io/hosting-and-deployment/hosting-on-github/

Hugo Book을 기준으로는 hugo.yaml에서 theme option을 추가해주면 된다.
```bash
run: |
          hugo \
          # add option -> --theme hugo-book \
            --theme hugo-book \
            --gc \
            --minify \
            --baseURL "${{ steps.pages.outputs.base_url }}/"
```