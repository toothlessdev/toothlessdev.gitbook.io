# NestJS TypeORM 연동하기

## 📖 ORM 이 뭐고 왜 사용하는걸까?

### ✏️ ORM ?

Object Relational Mapper (객체 관계 매핑)의 약자로, 객체와 관계형 데이터베이스의 데이터를 자동으로 매핑(연결) 해주는 기술입니다.

OOP 에서는 클래스를 사용하고, 관계형 데이터베이스에서는 테이블을 사용하기 때문에 서로 불일치가 존재하는데, ORM 을 사용하면 객체를 통해 간접적으로 관계형 데이터베이스의 데이터를 다룰 수 있습니다.



## 📖 NestJS TypeORM 연동하기

### ✏️ 의존성 패키지 설치

```sh
npm install @nestjs/typeorm typeorm
# 또는 yarn add @nestjs/typeorm typeorm
```

NestJS 에서 TypeORM 을 사용할 수 있게 만들어 줄 @nestjs/typeorm 패키지와 typeorm 패키지를 설치하고\
데이터베이스에 맞는 패키지를 설치해줍니다

```bash
npm install mysql2 # MySQL
npm install pg # PostgreSQL
# ...
```



### ✏️ app.module.ts 에 TypeORM 모듈 추가

`TypeOrmModule.forRoot()` 를 활용해서 TypeOrm 과 데이터베이스를 연결 할 수 있습니다.

1. PostgreSQL

```typescript
import { Module } from '@nestjs/common';
import { AppController } from './app.controller';
import { AppService } from './app.service';
import { TypeOrmModule } from '@nestjs/typeorm';

const TypeOrmRootModule = TypeOrmModule.forRoot({
    type: 'postgres', // 관계형 데이터베이스 타입
    host: '127.0.0.1',
    port: 5432,
    username: 'postgres',
    password: 'postgres',
    database: 'postgres',
    entities: [], // 생성할 모델 / 엔티티
    
    logging: true, // ORM 사용시 로그 남길지 여부
    autoLoadEntities: true, // entity파일 자동 로드 여부
    dropSchema: true, // 구동시 해당 테이블 삭제 여부 synchronize와 함께 사용 
    synchronize: true, // NestJS TypeORM 코드와 DB 를 동기화 (production 환경에서는 false 로)
});

@Module({
    imports: [TypeOrmRootModule],
    controllers: [AppController],
    providers: [AppService],
})
export class AppModule {}
```

2. SQLite

```typescript
import { Module } from '@nestjs/common'; 
import { AppController } from './app.controller'; 
import { AppService } from './app.service'; 
import { TypeOrmModule } from '@nestjs/typeorm'; 

const TypeORMRootModule = TypeOrmModule.forRoot({
    type: 'sqlite', // 관계형 데이터베이스 타입
    database: 'database.db', // DB 파일 이름 
    entities: [], // 생성할 모델 / 엔티티
    
    logging: true, // ORM 사용시 로그 남길지 여부
    autoLoadEntities: true, // entity파일 자동 로드 여부
    dropSchema: true, // 구동시 해당 테이블 삭제 여부 synchronize와 함께 사용 
    synchronize: true, // NestJS TypeORM 코드와 DB 를 동기화 (production 환경에서는 false 로)
}),

@Module({ 
    imports: [TypeOrmRootModule], 
    controllers: [AppController], 
    providers: [AppService], 
}) 

export class AppModule {}
```

다른 관계형 데이터베이스도 위와 같이 설정해주면 됍니다.



## 📖 Repository 패턴 사용하기

### ✏️ @Entity로 테이블 생성하기

@Entity() 어노테이션 / 데코레이터를 사용하면, 선언한 클래스를 기반으로 테이블이 생성됍니다.

```typescript
import { Entity, Column } from "typeorm";

@Entity()
export class MyModel {
    @Column()
    // column 생성 ...
}
```

이후, `TypeOrmModule.forRoot()` 의 entities 에 모델을 추가해 줍니다.

```typescript
const TypeORMRootModule = TypeOrmModule.forRoot({
    // ...
    entities: [MyModule], // 생성할 모델 / 엔티티
})
```



### ✏️ 모듈에 Repository 주입하기

`TypeOrmModule.forFeature()` 를 활용해서 모델에 해당하는 Repository 를 주입 할 수 있습니다.

```typescript
import { Module } from '@nestjs/common';
import { MyService } from './my.service';
import { MyController } from './my.controller';
import { TypeOrmModule } from '@nestjs/typeorm';
import { MyModel } from './entities/my.entity';

@Module({
    imports: [TypeOrmModule.forFeature([MyModel])],
    controllers: [MyController],
    providers: [MyService],
})
export class MyModule {}
```



### ✏️ Service에 Repository 주입하기

```typescript
import { Injectable } from '@nestjs/common';
import { Repository } from 'typeorm';
import { MyModel } from './entities/my.entity';
import { InjectRepository } from '@nestjs/typeorm';

@Injectable()
export class MyService {
    constructor(
        @InjectRepository(MyModel)
        private readonly myRepository: Repository<MyModel>,
    ) {}
}
```

Service 의 생성자에 주입받을 repository 를 만들고, \
`@InjectRepository` 로 NestJS IOC 컨테이너가 Repository를 주입 할 수 있도록 설정 해 주면 됍니다.



## 🔗 참고자료

NestJS - Database\
[https://docs.nestjs.com/techniques/database](https://docs.nestjs.com/techniques/database)

NestJS - SQL TypeORM\
[https://docs.nestjs.com/recipes/sql-typeorm](https://docs.nestjs.com/recipes/sql-typeorm)

