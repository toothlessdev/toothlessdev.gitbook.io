# NextJS SSG 정적사이트로 GithubPages 연동하기

NextJS 에서 정적 사이트 생성 SSG 를 이용해서 GithubPages 로 배포

## 📖 NextJS 프로젝트 생성하고 설정하기

### ✏️NextJS 프로젝트 생성

`npx create-next-app` 을 이용해서 NextJS 프로젝트를 생성합니다.

### ✏️next.config.mjs 수정

`next.config.mjs` 에서 nextConfig 을 다음과 같이 수정해줍니다.

```javascript
const nextConfig = {
    output: "export",
    basePath: "",
    images: {
        unoptimized: true,
    },
}
export default nextConfig;
```

NextJS 에서 jsx 로 작성된 파일을 이용해서 정적 사이트를 생성하기위해서 `output: "export"` 를 추가합니다

basePath 에는 GithubPages 배포시 URL 의 Host 뒷부분에 Path 를 추가해줍니다.\
(저는 https://toothlessdev.github.io/" 에 배포하기 위해서 빈 문자열을 추가했습니다)

```
Image Optimization using the default loader is not compatible with export.
  Possible solutions:
    - Use `next start` to run a server, which includes the Image Optimization API.
    - Configure `images.unoptimized = true` in `next.config.js` to disable the Image Optimization API.
  Read more: https://nextjs.org/docs/messages/export-image-api
```

NextJS 를 이용해 배포, 정적 사이트를 생성시, 이미지 최적화가 적용되지 않는다는 오류를 해결하기 위해 `images : { unoptimized : true }` 를 추가합니다.



### ✏️/public 디렉토리에 .nojekyll 파일 추가

GithubPage 가 기본적으로 Jekyll 로 정적사이트 생성을 방지하기 위해 public 디렉토리에 .nojekyll 파일을 추가합니다



## 📖 Base URL 추가하기

배포된 페이지에서 이미지 경로는 `https://~.github.io~` 가 되기 때문에 정상적으로 참조하기 위해서는 baseUrl 을 추가해주어야 한다

`/src/config/baseUrl.ts` 에

```typescript
export const baseUrl = process.env.NODE_ENV === "production" ? "http://~.github.io/" : "";
```

를 추가해주고, Images 앞에 baseUrl 을 추가해준다.

```tsx
<Image
    src={prefix + "/vercel.svg"}
    alt="Vercel Logo"
    className={styles.vercelLogo}
    width={100}
    height={24}
    priority 
/>
```



## 📖 Github Action Workflow 파일 추가

`.github/workflows/deploy.yml` 파일을 작성합니다.

```yaml
# Sample workflow for building and deploying a Next.js site to GitHub Pages
#
# To get started with Next.js see: https://nextjs.org/docs/getting-started
#
name: Deploy Next.js site to Pages

on:
  push:
    branches: ["main"]
  pull_request:
    branches: ["release"]
    types: ["closed"]
  

  # Allows you to run this workflow manually from the Actions tab
  workflow_dispatch:

# Sets permissions of the GITHUB_TOKEN to allow deployment to GitHub Pages
permissions:
  contents: read
  pages: write
  id-token: write

# Allow only one concurrent deployment, skipping runs queued between the run in-progress and latest queued.
# However, do NOT cancel in-progress runs as we want to allow these production deployments to complete.
concurrency:
  group: "pages"
  cancel-in-progress: false

jobs:
  # Build job
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Detect package manager
        id: detect-package-manager
        run: |
          if [ -f "${{ github.workspace }}/yarn.lock" ]; then
            echo "manager=yarn" >> $GITHUB_OUTPUT
            echo "command=install" >> $GITHUB_OUTPUT
            echo "runner=yarn" >> $GITHUB_OUTPUT
            exit 0
          elif [ -f "${{ github.workspace }}/package.json" ]; then
            echo "manager=npm" >> $GITHUB_OUTPUT
            echo "command=ci" >> $GITHUB_OUTPUT
            echo "runner=npx --no-install" >> $GITHUB_OUTPUT
            exit 0
          else
            echo "Unable to determine package manager"
            exit 1
          fi

      - name: Setup Node
        uses: actions/setup-node@v4
        with:
          node-version: "lts/*"
          cache: ${{ steps.detect-package-manager.outputs.manager }}

      - name: Setup Pages
        uses: actions/configure-pages@v4

      - name: Restore cache
        uses: actions/cache@v4
        with:
          path: |
            .next/cache
          # Generate a new cache whenever packages or source files change.
          key: ${{ runner.os }}-nextjs-${{ hashFiles('**/package-lock.json', '**/yarn.lock') }}-${{ hashFiles('**.[jt]s', '**.[jt]sx') }}
          # If source files changed but packages didn't, rebuild from a prior cache.
          restore-keys: |
            ${{ runner.os }}-nextjs-${{ hashFiles('**/package-lock.json', '**/yarn.lock') }}-

      - name: Install dependencies
        run: ${{ steps.detect-package-manager.outputs.manager }} ${{ steps.detect-package-manager.outputs.command }}

      - name: Build with Next.js
        run: ${{ steps.detect-package-manager.outputs.runner }} next build

      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: ./out

  # Deployment job
  deploy:
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    needs: build
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```



## 🔗 참고자료

[https://github.com/gregrickaby/nextjs-github-pages](https://github.com/gregrickaby/nextjs-github-pages)
