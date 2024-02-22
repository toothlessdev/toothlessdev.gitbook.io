# 관계형 데이터베이스와 TypeORM Column

## 📖 Column Annotation

### ✏️ @Column()

기본적인 관계형 데이터베이스에서의 Attribute (Column), 열을 매핑하는 Annotation 입니다.

### ✏️ @PrimaryColumn()

관계형 데이터베이스의 Relation (Table) 에서 Entity 를 유일하게 식별할 수 있게 하는 Attribute (Column) 를 Key Attribute 라고 합니다.

여러개의 Key Attribute 가 존재 할 수 있는데, 그 중 선택된 키를 Primary Key (기본키) 라고 하고,\
@PrimaryColumn() 어노테이션을 사용하면 해당 Attribute (Column) 를 Primary Key 로 매핑합니다.

### ✏️ @PrimaryGeneratedColumn()

관계형 데이터베이스에서 Tuple (Row) 이 생성될 때, 자동으로 Primary Key 에 해당하는 Key Attribute (Primary Column) 을 생성할 때 (auto-increment / sequence / serial ... ) @PrimaryGeneratedColumn() 을 사용할 수 있습니다.

```typescript
@PrimaryGeneratedColumn()
id: number;
```

파라미터로 아무런 값을 넣지 않으면 데이터가 생성될 때 마다 증가하는 정수값이 할당됩니다.

```typescript
@PrimaryGeneratedColumn("uuid")
id: string;
```

파라미터로 "uuid" 를 넣으면, 고유하게 식별가능한 UUID 가 할당됩니다

### ✏️ @CreateDateColumn()

```typescript
@CreateDateColumn()
createdAt: Date;
```

데이터가 생성되는 날짜와 시간이 자동으로 할당됩니다.

### ✏️ @UpdateDateColumn()

```typescript
@UpdateDateColumn()
updatedAt: Date;
```

데이터가 업데이트 되는 날짜와 시간이 자동으로 할당됩니다.

### ✏️ @VersionColumn()

```typescript
@VersionColumn()
version: number;
```

처음 생성될때는 1의 값을 갖고, 데이터가 업데이트 될 때 마다 1씩 증가합니다.\
(save 가 호출된 횟수가 됩니다)

### ✏️ @Column() @Generated()

Tuple (Row) 가 생성될 때 마다 해당 Column 의 값이 자동으로 할당됩니다.

```typescript
@Column()
@Generated("increment")
rowNo: number;
```

Tuple (Row) 가 생성될 때 마다 1씩 증가하는 정수값이 할당됩니다

```typescript
@Column()
@Generated("uuid")
rowUUID: string;
```

Tuple (Row) 가 생성될 때 마다 고유하게 식별가능한 UUID 가 할당됩니다.



## 📖 Column Property

`@Column()` 은 관계형 데이터베이스 Relation (Table) 에서 Attribute (Column) 과 자동으로 매핑해줍니다.\
[https://typeorm.io/entities#column-options](https://typeorm.io/entities#column-options)

### ✏️ Column Type

TypeORM 에서는 `@Entity()` 클래스에 정의해둔 타입을 기반으로 Attribute (Column) 의 타입을 결정합니다.\
`@Column({ type : "" })` 의 파라미터를 이용해 데이터베이스의 타입을 직접 지정할 수 있습니다

[https://typeorm.io/entities#column-types](https://typeorm.io/entities#column-types)

```typescript
@Column({
    type: "" // int, varchar ...
})
```

**✓ Enum Type**\
Attribute (Column) 에 Enum 타입을 지정하려는 경우 type 으로 "enum" 을 넣고, enum Property 에 enum 을 지정해줄 수 있습니다.\
Enum Type 을 지정하는 경우, Tuple (Row) 를 생성할때 열거형을 제외한 값이 들어오는 것을 방지 할 수 있어 무결성을 보장할 수 있습니다.

```typescript
enum MyEnum { 
    // ...
}
@Column({
    type: "enum",
    enum: MyEnum,
});
attr: MyEnum
```

### ✏️ Column Name

Relation (Table) 에서 실제 관계형 데이터베이스에서 Attribute (Column) 의 이름을 직접 지정할 수 있습니다.

```typescript
@Column({
    name: " "
})
```

### ✏️ length

정수값으로 Attribute (Column) 의 길이를 지정할 수 있습니다.

```typescript
@Column({
    length: 
})
```

### ✏️ update

```typescript
@Column({
    update: true / false
})
```

Boolean 값으로 true 일시, 처음 Tuple (Row) 를 저장할때만 값을 할당 할 수 있습니다.

### ✏️ nullable

```typescript
@Column({
    nullable: true / false
})
```

Boolean 값으로 해당 Attribute (Column) 이 Null 값을 가질 수 있는지 지정할 수 있습니다.

> 참고 : RDBMS 에서 NULL 값은
>
> 1. Unknown (값을 모름)
> 2. Not Available (값이 채워질 수 있음)
> 3. InApplicable (적용 불가능)
>
> 의 의미로 사용됩니다.

### ✏️ select

```typescript
@Column({
    select: true / false
})
```

Boolean 값으로 기본값이 true 입니다. find() 를 호출할때 기본으로 값을 불러올지 지정할 수 있습니다.\
select 가 false 인 Attribute (Column) 을 가져오려면, find 호출시 select 옵션을 넣어주어야 합니다.

```typescript
this.myRepository.find({
    select: {
        attributeName: true
    }
})
```

### ✏️ default

해당 Attribute (Column) 에 아무런 값도 할당하지 않은 경우 기본적으로 들어가는 값을 지정할 수 있습니다.

```typescript
@Column({
    default: // DEFAULT VALUE
})
```

### ✏️ unique

해당 Attribute (Column) 이 유일한 값이 되어야 할 경우,\
즉, Candidate Key 가 되어야 하는 경우 unique : true 을 통해 설정 할 수 있습니다

```typescript
@Column({
    unique: true / false
})
```

