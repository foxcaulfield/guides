1. Install nestjs
2. Create project folder
3. Create nested folder for server in it by initializing a nestjs project

```shell
nest new server
```

4. Install additional dependencies

```shell
npm i @nestjs/typeorm typeorm pg @nestjs/config
```

5. Create the **.env** file

```shell
touch .env
```

6. Fill in the **.env** file

```js
POSTGRES_HOST=127.0.0.1
POSTGRES_PORT=5432
POSTGRES_USER=postgres
POSTGRES_PASSWORD=password
POSTGRES_DATABASE=treesn_db
```

7. Set up configs in **app.module.ts**

```ts
/* module */
// ...
@Module({
    imports: [
        ConfigModule.forRoot({isGlobal: true}),
        TypeOrmModule.forRoot({
            type: "postgres",
            host: process.env.POSTGRES_HOST,
            port: parseInt(<string>process.env.POSTGRES_PORT, 10),
            username: process.env.POSTGRES_USER,
            password: process.env.POSTGRES_PASSWORD,
            database: process.env.POSTGRES_DATABASE,
            autoLoadEntities: true,
            synchronize: true, // only for development
        })
    ]
})
// ...
```

<!-- 8. Create a feed module

```shell
nest generate module feed
```

9. Create a feed service

```shell
nest generate service feed/services/feed --flat --no-spec
```

10. Create a feed controller

```shell
nest generate controller feed/controllers/feed --flat --no-spec
``` -->

8. 9. 10. Create a feed resource

```shell
nest generate resource feed
```

11. Set the global prefix for the app

```ts
/* main.ts */
// ...
app.setGlobalPrefix("api");
// ...
```

12. Create a model

```shell
mkdir src/feed/models
touch src/feed/models/post.interface.ts
```

13. Fill it in with this:

```ts
/* interface */
export interface IPost {
  id?: number;
  body?: string;
  createdAt?: Date;
}
```

14. Crete an entity:

```shell
touch src/feed/entities/post.entity.ts
```

15. Fill it in with that _(consider an import from TypeORM)_:

```ts
/* entity */
import { Column, Entity, PrimaryGeneratedColumn } from "typeorm";
import { IPost } from "../models/post.interface";

@Entity("feed_post")
export class PostEntity implements IPost {
  @PrimaryGeneratedColumn() // primary key
  id: number;

  @Column({ default: "" })
  body: string;

  @Column({
    type: "timestamp",
    default: () => "CURRENT_TIMESTAMP",
  })
  createdAt: Date;
}
```

16. Import TypeORM module to _feed.module.ts_

```ts
/* module */
// ...
@Module({
  imports: [TypeOrmModule.forFeature([FeedPostEntity])],
  // ...
})
export class FeedModule {}
// ...
```

17. Inject the **FeedPostEntity** to **FeedService** in order to
    implement a repository pattern

18. Inject the feed service **to** the feed **controller** with the constructor based injection

19. Import **FeedController** to **FeedModule** imports array (??? it is in controllers)

20. feed.service.ts content

```ts
import { Injectable } from "@nestjs/common";
import { InjectRepository } from "@nestjs/typeorm";
import { PostEntity } from "./entities/post.entity";
import { DeleteResult, Repository, UpdateResult } from "typeorm";
import { IPost } from "./models/post.interface";
import { Observable, from } from "rxjs";

@Injectable()
export class FeedService {
  constructor(
    @InjectRepository(PostEntity)
    private readonly postRepository: Repository<PostEntity>
  ) {}

  createPost(postDto: IPost): Observable<IPost> {
    return from(this.postRepository.save(postDto));
  }

  findAllPosts(): Observable<Array<IPost>> {
    return from(this.postRepository.find());
  }

  // findOnePost(id: number) {
  //     return `This action returns a #${id} feed`;
  // }

  updatePost(id: number, postDto: IPost): Observable<UpdateResult> {
    return from(this.postRepository.update(id, postDto)); // partial data
  }

  removePost(id: number): Observable<DeleteResult> {
    return from(this.postRepository.delete(id));
  }
}
```

21. feed.controller.ts content

```ts
import {
  Controller,
  Get,
  Post,
  Body,
  Patch,
  Param,
  Delete,
} from "@nestjs/common";
import { FeedService } from "./feed.service";
import { IPost } from "./models/post.interface";
import { Observable } from "rxjs";
import { DeleteResult, UpdateResult } from "typeorm";

@Controller("feed")
export class FeedController {
  constructor(private readonly feedService: FeedService) {}

  @Post()
  create(@Body() postDto: IPost): Observable<IPost> {
    return this.feedService.createPost(postDto);
  }

  @Get()
  findAll(): Observable<Array<IPost>> {
    return this.feedService.findAllPosts();
  }

  // @Get(":id")
  // findOne(@Param("id") id: string) {
  //     return this.feedService.findOne(+id);
  // }

  @Patch(":id")
  update(
    @Param("id") id: string, // this is actually a string; don't know, there was the pipes of converting somewhere
    @Body() postDto: IPost
  ): Observable<UpdateResult> {
    return this.feedService.updatePost(+id, postDto);
  }

  @Delete(":id")
  remove(@Param("id") id: string): Observable<DeleteResult> {
    return this.feedService.removePost(+id);
  }
}
```
