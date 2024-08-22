# SRP 원칙으로 리팩토링 하기

## 📖 컴포넌트

<figure><img src="../../.gitbook/assets/image (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

<div align="center">

<figure><img src="../../.gitbook/assets/image (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

</div>

Save My Receipt 는 특정 그룹에 가입하거나 생성하여 해당 그룹에 영수증을 업로드 하고, 회계 관리자는 실물 영수증을 손쉽게 관리할 수 있는 서비스입니다.

'그룹'에 해당하는 정보는 'GroupCard' 라는 컴포넌트로 보여지고, \
검색시에는 '가입하기' 버튼이, 마이페이지에서는 '탈퇴하기' 버튼이 존재합니다.

둘다 디자인과 보여줄 데이터가 유사하고, 버튼에 해당하는 기능만 다르기 때문에 하나의 컴포넌트로 만들 수 있을 것입니다



## 📖 기존 코드

컴포넌트를 만들때, `variants` 라는 `props` 를 받아, 해당하는 값이 `SEARCH` 인 경우 검색화면에서 보여줄 녹색 버튼과 가입 API 호출을 핸들링 하는 이벤트 리스너를 바인딩 해 주었습니다.

`SEARCH` 가 아닌 경우, 빨강 버튼과 탈퇴 API 호출을 핸들링 하는 이벤트 리스너를 바인딩 해 주었습니다

```tsx
export const GroupCard: React.FC<IGroupCard> = ({
    id,
    name,
    city,
    organization,
    description,
    memberCount,
    receiptCount,
    accountantName,
    variants,
}) => {
    const router = useRouter();

    const handleJoinBtnClick = useCallback(async () => {
        const request = groupsService.joinGroup(id);
        toast
            .promise(request, {
                pending: "그룹 가입중입니다",
                success: "그룹 가입 성공!",
                error: "이미 가입된 그룹입니다",
            })
            .then(() => {
                router.push({ pathname: `/groups/${id}` });
            });
    }, [id, router]);

    const handleLeaveBtnClick = useCallback(async () => {
        const request = groupsService.leaveGroup(id);
        toast
            .promise(request, {
                success: "그룹 탈퇴 완료!",
                pending: "잠시만 기다려주세요..",
                error: "그룹 탈퇴 실패! 관리자에게 문의해주세요",
            })
            .then(() => {
                queryClient.invalidateQueries({ queryKey: [`/groups`] });
            });
    }, [id]);

    return (
        <Card>
            <CardHeader>
                <div className="flex justify-between items-center mb-2">
                    <div className="text-xs text-gray-500 overflow-hidden text-ellipsis">
                        {city}/{organization}
                    </div>
                    <div className="text-xs text-gray-500 flex-shrink-0">
                        <span>
                            <FontAwesomeIcon icon={faUsers} className="mx-1" />
                            {memberCount} 명
                        </span>
                        <span>
                            <FontAwesomeIcon icon={faReceipt} className="mx-1" />
                            {receiptCount || "0"} 개
                        </span>
                    </div>
                </div>
                <div className="flex items-center gap-2">
                    <div className="text-xl p-0 font-semibold overflow-hidden text-ellipsis">{name}</div>
                </div>
            </CardHeader>
            <CardContent>{description}</CardContent>
            <CardFooter className="flex justify-between items-center">
                <div className="flex items-center gap-2">
                    <Avatar className="w-6 h-6 border">
                        <AvatarImage alt="Avatar" src="/placeholder-user.jpg" />
                    </Avatar>
                    <div className="text-xs text-gray-500">{accountantName}</div>
                </div>
                <Button
                    size="sm"
                    variant={variants === "SEARCH" ? "default" : "destructive"}
                    onClick={variants === "SEARCH" ? handleJoinBtnClick : handleLeaveBtnClick}
                >
                    {variants === "SEARCH" ? "가입하기" : "탈퇴하기"}
                </Button>
            </CardFooter>
        </Card>
    );
};
```



## 📖 Single Responsibility 관점에서 본 기존코드

전체 서비스에서 총 두가지의 형태로 그룹 카드 컴포넌트가 사용된다고 하더라도,\
추후에 또 다른 기능으로 확장 될 수 도 있기 때문에

해당 코드는 SOLD 객체지향 설계 원칙 중, Single Responsibility 에 어긋납니다\
한 컴포넌트가 두가지의 기능을 담당하고 있기 때문입니다

따라서, 그룹에 대한 정보만 보여주는 `GroupCard` 컴포넌트를 만들고,\
해당 `GroupCard` 컴포넌트에 버튼과 이벤트 핸들러를 바인딩해주는 각각의\
`GroupJoinedCard` 와 `GroupSearchCard` 로 분리 해 주어야 합니다



## 📖 수정된 코드

```tsx
export interface IGroupCard extends ISearchGroup, Omit<React.ComponentProps<"div">, "id"> {
    button: React.ReactNode;
}
```

먼저 `id, name, city, organization, description, memberCount, receiptCount, accountantName` 를 가지고 있는 `ISearchGroup` 인터페이스와 확장성에 대비해 `Card` 컴포넌트 즉 div 엘리먼트에 들어갈 `ComponentProps<"div">` 를 확장한 인터페이스를 만들어 주었습니다

버튼 엘리먼트의 스타일이 다르기 때문에, `React.ReactNode` 타입의 button 을 받아 표시할 수 있도록 구성했습니다.

```tsx
export const GroupCard: React.FC<IGroupCard> = ({
    id,
    name,
    city,
    organization,
    description,
    memberCount,
    receiptCount,
    accountantName,
    button,
    ...rest
}) => {
    return (
        <Card {...rest}>
            <CardHeader>
                <div className="flex justify-between items-center mb-2">
                    <div className="text-xs text-gray-500 overflow-hidden text-ellipsis">
                        {city}/{organization}
                    </div>
                    <div className="text-xs text-gray-500 flex-shrink-0">
                        <span>
                            <FontAwesomeIcon icon={faUsers} className="mx-1" />
                            {memberCount} 명
                        </span>
                        <span>
                            <FontAwesomeIcon icon={faReceipt} className="mx-1" />
                            {receiptCount || "0"} 개
                        </span>
                    </div>
                </div>
                <div className="flex items-center gap-2">
                    <div className="text-xl p-0 font-semibold overflow-hidden text-ellipsis">{name}</div>
                </div>
            </CardHeader>
            <CardContent>{description}</CardContent>
            <CardFooter className="flex justify-between items-center">
                <div className="flex items-center gap-2">
                    <Avatar className="w-6 h-6 border">
                        <AvatarImage alt="Avatar" src="/placeholder-user.jpg" />
                    </Avatar>
                    <div className="text-xs text-gray-500">{accountantName}</div>
                </div>
                {button}
            </CardFooter>
        </Card>
    );
};
```

이후 해당 컴포넌트에 `<Button>` 컴포넌트가 들어갈 곳에 `{button}` props 를 넣어 주었습니다



```tsx
export const GroupSearchCard: React.FC<IGroupSearchCard> = ({ button, ...props }) => {
    const router = useRouter();

    const handleClick = useCallback(async () => {
        const request = groupsService.joinGroup(props.id);
        toast
            .promise(request, {
                pending: "그룹 가입중입니다",
                success: "그룹 가입 성공!",
                error: "이미 가입된 그룹입니다",
            })
            .then(() => {
                router.push({ pathname: `/groups/${props.id}` });
            });
    }, [props.id, router]);

    return (
        <GroupSearchCard
            {...props}
            button={
                <Button size="sm" variant="default" onClick={handleClick}>
                    가입하기
                </Button>
            }
        />
    );
};
```

이후, 그룹 검색 페이지에 보여줄 `GroupSearchCard` 컴포넌트를 만들고,\
녹색 버튼과 클릭시 이벤트 핸들러를 연결해 주었습니다.



