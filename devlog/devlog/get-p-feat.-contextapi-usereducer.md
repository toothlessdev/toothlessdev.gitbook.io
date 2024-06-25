# \[GET-P] 기술스택 선택 컴포넌트 구현 (feat. ContextAPI, useReducer)

이번에 진행하는 GET-P 프로젝트에서 다음과 같은 컴포넌트의 구현을 맡게 되었다\
컴포넌트의 요구사항은 다음과 같다

<figure><img src="../../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

* 상단에는 검색바가 존재하고, 검색시에, 하위 기술슽택이 존재하는 아코디언 컨테이너가 열린다
* 아코디언 컨테이너를 누르면 색상이 변경되고, 컨테이너가 열려, 하위 아이템들이 보여진다
* 아코디언 하위 아이템을 클릭하면, 우측에 기술스택 뱃지가 등록되고 파란색으로 하이라이트 처리된다
* 이미 선택된 하위 아이템 클릭시 우측 기술스택 뱃지가 제거된다
* 우측 선택된 기술스택 목록 컨테이너에 X 버튼 클릭시 우측 기술스택 뱃지가 제거된다



컴포넌트를 크게 4개로 나누어 보았다

* 기술 검색바 컴포넌트
* 아코디언 컴포넌트
* 기술스택 뱃지 컴포넌트
* 아코디언 Wrapper 컴포넌트



### ✏️ Accordion 컴포넌트

<figure><img src="../../.gitbook/assets/image (1) (1).png" alt=""><figcaption></figcaption></figure>

기본적으로 Accordion 컴포넌트는 css 의 `max-height` 프로퍼티 값을 변경해서 기능을 구현한다\
초기 `max-height` 를 버튼 한칸 높이로 정해두고, Accordion 이 클릭되었을때 `max-height` 를 버튼 한칸 높이 X 하위 버튼요소 개수 높으로 설정하면 된다

```tsx
export const TechStackAccordion: React.FC<ITechStackAccordionGroup> = ({ width, groupName, groupItems }) => {
    const containerRef = useRef<HTMLDivElement>(null);
    const buttonRef = useRef<HTMLButtonElement>(null);
    const iconRef = useRef<HTMLImageElement>(null);

    const [isOpened, setIsOpened] = useState<boolean>(false);

    const { state, dispatch } = useTechStack();

    const handleClick = useCallback(() => {
        setIsOpened((isOpened) => !isOpened);
    }, []);

    const handleItemClick = useCallback(
        (e: React.MouseEvent<HTMLDivElement>) => {
            dispatch({
                type: "TOGGLE_TECH_STACK",
                payload: {
                    value: e.currentTarget.innerText,
                },
            });
        },
        [dispatch],
    );

    useEffect(() => {
        if (containerRef.current && buttonRef.current && iconRef.current) {
            if (isOpened) {
                containerRef.current.style.maxHeight = `${groupItems.length * 56}px`;
                buttonRef.current.style.backgroundColor = `#F8F6F8`;
                iconRef.current.style.transform = `rotate(180deg)`;
            } else {
                containerRef.current.style.maxHeight = `0px`;
                buttonRef.current.style.backgroundColor = `#fff`;
                iconRef.current.style.transform = `rotate(0deg)`;
            }
        }
    }, [isOpened, groupItems.length]);

    return (
        <TechStackAccordionWrapper width={width}>
            <TechStackAccordionButton ref={buttonRef} onClick={handleClick}>
                <AccordionItemText>{groupName}</AccordionItemText>
                <Chevron ref={iconRef} src={chevronIcon}></Chevron>
            </TechStackAccordionButton>

            <TechStackAccordionContainer ref={containerRef}>
                {groupItems.map((item) => {
                    return (
                        <TechStackAccordionItem
                            isSelected={state.selected.findIndex((selectedItem) => selectedItem.value === item) !== -1}
                            onClick={handleItemClick}
                        >
                            {item}
                        </TechStackAccordionItem>
                    );
                })}
            </TechStackAccordionContainer>
        </TechStackAccordionWrapper>
    );
};

```



## 📖 컴포넌트 구현

### ✏️ TechStackBadge 컴포넌트

<figure><img src="../../.gitbook/assets/image (2) (1).png" alt=""><figcaption></figcaption></figure>



두 컴포넌트를 각각 만들고, 해당 컴포넌트들을 사용하는 TechStackSelector 컴포넌트도 만들어주었다

```tsx
export const TechStackSelector: React.FC<ITechStackSelector> = ({ width, height, techStack }) => {
    const handleChange = useCallback((e: ChangeEvent<HTMLInputElement>) => {
        console.log(e.target.value);
    }, []);

    return (
        <TechStackSelectorWrapper width={width}>
            <SearchIcon src={searchIcon} />
            <TechStackSearchInput placeholder="기술 검색" onChange={handleChange} />

            <TechStackSelectorContainer height={height}>
                <TechStackSelectorGroupContainer>
                    {techStack.map((stack) => {
                        return (
                            <TechStackAccordion
                                width="100%"
                                groupName={stack.groupName}
                                groupItems={stack.groupItems}
                            ></TechStackAccordion>
                        );
                    })}
                </TechStackSelectorGroupContainer>

                <SelectedTechStacks>
                    {state.selected.map((techStack) => {
                        return <TechStackBadge text={techStack.value} />;
                    })}
                </SelectedTechStacks>
            </TechStackSelectorContainer>
        </TechStackSelectorWrapper>
    );
};

```



## 📖 ContextAPI vs Redux

TechStackAccordion 컴포넌트에서 각 하위 아이템들을 클릭할때 마다 선택한 기술스택이 TechStackBadge 로 렌더링되어야 하기 때문에, 각 컴포넌트 사이 공유되는 상태가 존재해야 한다

전역 상태 관리 라이브러리고 Redux Toolkit 을 사용하고 있는데,\
"기술스택선택 컴포넌트" 내부에서만 관리되는 상태이기 때문에 Context API 를 사용하였다



## 📖 ContextAPI + useReducer

처음에 useState 를 Context 로 전달해 각 하위 컴포넌트에서 사용할 수 있도록 설계 하였는데,\
상태에 대해 변경하려는 액션 타입이 정해져 있고 (삭제, 추가, 토글)\
해당 액션을 처리하는 로직이 복잡할것으로 생각되어 useReducer 를 함께 사용하도록 설계하였다

Redux 의 Flux 아키텍쳐에 대해 알고 있다면 useReducer 사용은 쉽다

### ✏️ Context 생성

```tsx
const techStackState: TechStackState = {
    selected: [],
};

const TechStackContext = createContext<{
    state: TechStackState;
    dispatch: React.Dispatch<TechStackAction>;
}>({
    state: techStackState,
    dispatch: () => undefined,
});

export interface TechStackItem {
    value: string;
}

export interface TechStackState {
    selected: TechStackItem[];
}
```

먼저 선택된 초기 기술스택에 대한 상태를 만들고, Context 로 초기 상태를 설정해준다



### ✏️ useReducer 를 사용한 상태관리 액션 정의

```tsx
export interface TechStackAction {
    type: "ADD_TECH_STACK" | "REMOVE_TECH_STACK" | "TOGGLE_TECH_STACK";
    payload: TechStackItem;
}
```

```tsx
const reducer: React.Reducer<TechStackState, TechStackAction> = (state, action) => {
    switch (action.type) {
        case "ADD_TECH_STACK":
            return {
                ...state,
                selected: [
                    ...state.selected,
                    {
                        value: action.payload.value,
                    },
                ],
            };
        case "REMOVE_TECH_STACK":
            return {
                ...state,
                selected: state.selected.filter((item) => item.value !== action.payload.value),
            };
        case "TOGGLE_TECH_STACK":
            const exists = state.selected.find((item) => item.value === action.payload.value);
            if (exists) {
                return {
                    ...state,
                    selected: state.selected.filter((item) => item.value !== action.payload.value),
                };
            } else {
                return {
                    ...state,
                    selected: [...state.selected, action.payload],
                };
            }
        default:
            return state;
    }
};
```

Union Type 으로 액션 타입을 지정해주고,\
해당 액션을 dispatch 하는 reducer 함수를 만들어 준다



### ✏️ Provider, hook 생성

```tsx
export const TechStackProvider = ({ children }: { children: React.ReactNode }) => {
    const [state, dispatch] = useReducer(reducer, techStackState);
    return <TechStackContext.Provider value={{ state, dispatch }}>{children}</TechStackContext.Provider>;
};

// eslint-disable-next-line react-refresh/only-export-components
export const useTechStack = () => {
    const context = useContext(TechStackContext);
    if (!context) throw new Error("useTechStack 은 TechStackProvider 내부에서 사용되어야 합니다");
    return context;
};
```

