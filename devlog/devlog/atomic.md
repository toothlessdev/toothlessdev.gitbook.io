# \[DEVLOG] Atomic 컴포넌트 디자인 패턴 도입기

리액트로 프로젝트를 할 때 마다 \
_**" 컴포넌트를 어떻게 나눠야 할까? "**_\
는 항상 하게 되는 고민 중 하나 인 것 같다.

그러다 카카오 개발블로그에 아토믹 디자인 패턴을 발견했다.

{% embed url="https://fe-developers.kakaoent.com/2022/220505-how-page-part-use-atomic-design-system/" %}

## 📖 Atomic 디자인 패턴이란?

Atomic 디자인 패턴은 컴포넌트를 가장 작은 단위,\
순수 HTML 요소 하나를 리턴하는 Atom 이라는 단위 부터\
Molecules, Organisms, Templates, Pages 로 재사용하면서 확장하는 컴포넌트 디자인 패턴이다.



예를들어, NavBar 라는 컴포넌트가 다음과 같을때,

```jsx
export const NavBar = () => {
    return (
        <nav>
            <ul>
                <li>
                    <Link>
                        <span></span>
                    </Link>
                </li>
            </ul>
        </nav>
    );
};
```

Atomic 디자인 패턴을 사용하면 다음과 같이 나눌 수 있다

```jsx
// Atom
export const Text = ({size, children}) => {
    return <span>{children}</span>;
}
// Molecule
export const NavItem = ({to, children}) => {
    return (
        <li>
            <Link to={to}>{children}</Link>;
        </li>
    );
}
// Organism
export const NavBar = ({children}) => {
    return (
        <nav>
            <ul>{children}</ul>  
        </nav>
    );
}
// Template
//...
```



이를 통해 재사용성이 올라가고, 컴포넌트를 분활하는데 도움을 준다.



## 📖 도입 실패

처음에는 어느정도 컴포넌트를 나누는 기준이 정해지고, 재사용성이 많이 올라갈것을 기대하고 Atomic 컴포넌트 디자인 패턴을 도입했지만.. 금세 어려움을 겪게 되었다.

우선 카카오 기술블로그에서는

> 1. **Molecule과 Organism 을 나누는 기준의 주관성**
> 2. #### **Organism을 나누는 기준** <a href="#organism" id="organism"></a>
> 3. #### **약간 다른 Organism이 있을 때 중복 컴포넌트가 생기거나 불필요한 Props의 증가** <a href="#organism-props" id="organism-props"></a>

정도의 한계가 있었다고 한다.



실제로 적용하면서 비슷한 문제를 겪기도 했고,\
다음은 내가 프로젝트에 Atomic 디자인 패턴을 사용하면서 느낀 단점이다..

> 1. 브랜치를 GitFlow 로 관리를 하고 있었기 때문에, 개발 초기에 컴포넌트가 파편화되어 cherry-pick 이 너무 많아졌다.&#x20;

그런데 이건 필요한 컴포넌트를 따로 개발하기 때문이 더 큰 것 같다.



> 2. 컴포넌트가 더욱 더 파편화 되는 것 같다

위에서 있던 예제와 같이 NavBar 컴포넌트를 구현했을때,

NavBar, NavItem ... 와 같이 NavBar 와 관련된 컴포넌트가 Atom, Molecules ... 에 흩어져 연관된 컴포넌트끼리의 응집도가 더 떨어지는 느낌을 받았다.



> 3. 불필요한 Props 의 증가, 복잡한 컴포넌트 내부 구현

Atomic 디자인 패턴이 재사용성을 높이는데 집중이 되어서 그런가? 컴포넌트를 만들때 최대한 재사용성을 높이려고 하다보니, props 가 점점 더 많아지고, 그에 따른 분기도 생겨 컴포넌트 내부가 점점 복잡해져가는 느낌을 받았다

(그냥 내 코드가 더러운 것일 수도 있다)



이런 이유들 때문에 Atomic 컴포넌트 디자인 패턴을 도입하려다 말았다..



## 📖 그렇다면 어떻게 나눠야 하나?

그렇다면 컴포넌트를 어떤 기준으로 나눠야 할까?

> 재사용이 가능하면서도, 내부 구현이 복잡해지지 않는 이상적인....

사실 정답은 없는것 같다. 특히나 리액트가 정해진 구조가 없는 그냥 컴포넌트 패턴을 적용한 라이브러리인것도 한몫 하는것 같다.

일단 재사용성을 높이려면 컴포넌트간의 의존성을 낮춰야하고... 이런저런 생각이 많이 드는데 \
이런 기준들은 코드를 많이 짜다보면 공통된 부분이 보이지 않을까 하고 생각한다







