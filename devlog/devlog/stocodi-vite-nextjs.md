# \[Stocodi] Vite 에서 NextJS 로 마이그레이션

## ✳️ 발단

아는 형이 '스코토디' 라는 금융교육 플랫폼 관련해서 스타트업을 진행하는데 개발자로 들어 와볼래 라는 제안을 받은지 4개월 정도가 지났다.

이번에 새로운 서비스를 기획중이기도하고, \
특히나 검색엔진 최적화, 업로드한 게시물도 검색엔진에 노출되었으면 좋겠다 .... 와 같은 요구사항이 늘어나게 되면서, 기존에 Vite 로 만든 리액트 애플리케이션을 NextJS Page Router 로 마이그레이션 하기로 했다



## ✳️ 마이그레이션

{% embed url="https://nextjs.org/docs/pages/building-your-application/upgrading/from-vite#step-1-install-the-nextjs-dependency" %}

1. Install NextJS Dependency : NextJS 의존성 패키지 설치하기

```
npm install next@latest
```

2. Create NextJS Configuration File : next.config.mjs 파일 생성하기

```javascript
/** @type {import('next').NextConfig} */
const nextConfig = {
  output: 'export', // Outputs a Single-Page Application (SPA).
  distDir: './dist', // Changes the build output directory to `./dist/`.
}
 
export default nextConfig
```

3.
