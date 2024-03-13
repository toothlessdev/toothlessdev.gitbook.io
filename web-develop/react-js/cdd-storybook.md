# CDD 와 StoryBook

## 📖 CDD



## 📖 Story

StoryBook 은 Story 라는 단위로 구성되어 있고, 컴포넌트는 하나 이상의 Story 로 구성됍니다.

Story 는 Component Story Format (CSF) 라는 포맷을 이용해 작성됍니다.



```typescript
import type { Meta } from "@storybook/~";
import { MyComponent } from "./MyComonent";

const meta : Meta<typeof MyComponent> = 

export default {
        
}
```









## 🔗 참고자료

[https://storybook.js.org/docs/api/csf](https://storybook.js.org/docs/api/csf)\
