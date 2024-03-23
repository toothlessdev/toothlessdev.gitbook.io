# CDD 와 StoryBook

## 📖 CDD





\--- storybook

First, import Meta and StoryObj for type safety and autocompletion in TypeScript stories.

Next, import a component. In this case, the Button component.



#### Meta

The default export, Meta, contains metadata about this component's stories. The title field (optional) controls where stories appear in the sidebar.



#### Args

Args are inputs that are passed to the component, which Storybook uses to render the component in different states. In React, args = props. They also specify the initial control values for the story.

\




#### Story

Each named export is a story. Its contents specify how the story is rendered in addition to other configuration options.

\




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
