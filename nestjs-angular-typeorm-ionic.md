<!-- # Nestjs init

1. Install nestjs
2. Create project folder
3. Create nested folder for server in it by initializing a nestjs project

```shell
nest new server
``` -->

<!-- 4. Install additional dependencies

```shell
npm i @nestjs/typeorm typeorm pg @nestjs/config
```

5. Create the **.env** file

```shell
touch .env
``` -->

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
  imports: [TypeOrmModule.forFeature([PostEntity])],
  // ...
})
export class FeedModule {}
// ...
```

17. Inject the **PostEntity** to **FeedService** in order to
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

22. Create init database script

```shell
touch init.sql
```

```sql
CREATE DATABASE treesn_db;
```

22. Create _docker-compose.file_

```yml
version: "3.8"
services:
  postgres_db:
    image: postgres
    container_name: postgres_cont
    restart: always
    env_file:
      - .env
    ports:
      - "9999:5432"
    volumes:
      - postgres_db:/var/lib/postgresql/data
      - ./init.sql:/docker-entrypoint-initdb.d/init.sql
      - ./init.sh:/docker-entrypoint-initdb.d/init.sh
    entrypoint: ["sh", "/docker-entrypoint-initdb.d/init.sh"]
volumes:
  postgres_db:
    driver: local
```

23. Init script for db exist check **init.sh**

```shell
#!/bin/bash
set -e

# Проверяем, существует ли база данных
if psql -U "$POSTGRES_USER" -tc "SELECT 1 FROM pg_database WHERE datname = '$POSTGRES_DATABASE'" | grep -q 1; then
    echo "Database $POSTGRES_DATABASE already exists"
else
    echo "Creating database $POSTGRES_DATABASE"
    psql -U "$POSTGRES_USER" -c "CREATE DATABASE $POSTGRES_DATABASE"
fi
```

```shell
chmod +x init.sh
```

# Angular init

- Install ionic cli

```shell
npm i -g @ionic/cli
```

- Init ionic app

```shell
ionic start
```

- Start ionic app from the proper directory

```shell
ionic serve
```

- Create a _header_ component

```shell
ionic generate component home/components/header
```

- Import the component (?)

```ts
// home.module.ts
//...
declarations: [HomePage, HeaderComponent];
// ...
```

- Delete everything from _home.page.html_
- Delete everything from _home.page.scss_
- Create a _popover_ component

```shell
ionic generate component home/components/header/popover
```

- Add a profile picture to _src/assets/profile-pic.jpg_

- Content of _header.component_

```html
<!-- header.component.html -->
<ion-header class="ion-container ion-no-border">
  <ion-toolbar>
    <div class="toolbar-wrapper">
      <ion-buttons slot="start" class="ion-no-padding">
        <ion-button>
          <ion-icon color="primary" class="treesn-logo" name="treesn-logo">
          </ion-icon>
        </ion-button>
        <ion-searchbar
          class="ion-hide-lg-down"
          color="tertiary"
          placeholder="Search"
        >
        </ion-searchbar>
      </ion-buttons>

      <ion-grid>
        <ion-row class="ion-justify-content-end">
          <ion-col size="auto">
            <ion-icon name="home"></ion-icon>
            <div>Home</div>
          </ion-col>
          <ion-col size="auto">
            <ion-icon name="People"></ion-icon>
            <div>My network</div>
          </ion-col>
          <ion-col size="auto">
            <ion-icon name="briefcase"></ion-icon>
            <div>Jobs</div>
          </ion-col>
          <ion-col size="auto">
            <ion-icon name="chatbox-ellipses"></ion-icon>
            <div>Messages</div>
          </ion-col>
          <ion-col size="auto">
            <ion-icon name="notifications"></ion-icon>
            <ion-badge
              color="danger"
              style="
              position: absolute;
              margin-left: -10px;
              margin-top: -5px;
              height: 16px;
              width: 16px;
              border-radius: 9999px;"
              >2</ion-badge
            >
            <div>Notifications</div>
          </ion-col>
          <ion-col size="auto" (click)="presentPopover($event)">
            <ion-avatar>
              <ion-img src="../../../../assets/profile-pic.jpg"></ion-img>
            </ion-avatar>
            <div
              style="
              display: flex;
              justify-content: center;
              align-items: center;
            "
            >
              Me
              <ion-icon
                style="font-size: 16px;"
                name="caret-down-outline"
              ></ion-icon>
            </div>
          </ion-col>
        </ion-row>
      </ion-grid>
    </div>
  </ion-toolbar>
</ion-header>
```

```scss
// header.component.scss
.toolbar-wrapper {
  display: flex;
  max-width: 1128px;
  margin: auto;
}

.treesn-logo {
  font-size: 40px;
}

ion-icon {
  font-size: 24px;
}

ion-col {
  width: 80px !important;
  text-align: center;
  div {
    font-size: 12px;
    font-weight: 400;
  }
}

ion-badge {
  font-size: 12px;
  font-weight: 400;
}

ion-searchbar {
  --box-shadow: none;
  transform: scale(0.8);
  margin-left: -40px;
}

ion-header {
  border-bottom: 1px solid var(--ion-color-light-tint);
}

ion-avatar {
  width: 24px;
  height: 24px;
  margin: auto;
}
```

```ts
// header.component.ts
import { Component, OnInit } from "@angular/core";
import { PopoverController } from "@ionic/angular"; // from docs
import { PopoverComponent } from "./popover/popover.component";

@Component({
  selector: "app-header"
  templateUrl: "./header.component.html"
  styleUrls: ["./header.component.scss"]
})
export class HeaderComponent implements OnInit {
  constructor(
    public popoverController: PopoverController // inject it
  ) {}

  ngOnInit() {  }

  async presentPopover(event: any) {
    const popover = await this.popoverController.create({
      component: PopoverComponent,
      cssClass: "my-custom-class",
      event: event,
      // translucent: true
      showBackdrop: false
    });
    await popover.present();

    const { role } = await popover.onDidDismiss();
    console.log("onDidDismiss resolved with role", role)
  }
}
```

```scss
// global.scss
.my-custom-class .popover-content {
  width: 300px;
  margin-top: 16px;

  --box-shadow: none;
}
```

- Content of _popover.component.html_

```html
<!-- popover.component.html -->
<ion-card>
  <ion-card-header
    ><ion-grid>
      <ion-row class="ion-align-items-center ion-justify-content-center">
        <ion-col size="auto">
          <ion-avatar>
            <ion-img src="../../../../../assets/profile-pic.jpg"></ion-img>
          </ion-avatar>
        </ion-col>
        <ion-col>
          <ion-card-title> The Name </ion-card-title>
          <ion-card-subtitle> Full Stack Developer </ion-card-subtitle>
        </ion-col>
      </ion-row>
    </ion-grid>
    <ion-button expand="block" size="small" fill="outline" color="primary"
      >View Profile</ion-button
    >
  </ion-card-header>

  <ion-card-content>
    <ion-card-subtitle color="dark">Account</ion-card-subtitle>
    <p class="ion-padding-top">Settings & Privacy</p>
    <p class="ion-padding-top">Help</p>
    <p class="ion-padding-top">Language</p>
  </ion-card-content>

  <div class="item-divider"></div>

  <ion-card-content>
    <ion-card-subtitle color="dark">Manage</ion-card-subtitle>
    <p class="ion-padding-top">Posts & Activities</p>
    <p class="ion-padding-top">Job Posting Account</p>
  </ion-card-content>

  <div class="item-divider"></div>

  <ion-card-content>
    <p (click)="onSignOut()">Sign out</p>
  </ion-card-content>
</ion-card>
```

```scss
// popover.component.scss
.item-divider {
  border-bottom: 1px solid var(--ion-color-light-tint);
}

ion-card-content p:hover {
  text-decoration: underline;
  cursor: pointer;
}
```

```ts
// popover.component.ts
import { Component, OnInit } from "@angular/core";

@Component({
  selector: "app-popover",
  templateUrl: "./popover.component.html",
  styleUrls: ["./popover.component.scss"],
})
export class PopoverComponent implements OnInit {
  constructor() {}
  ngOnInit() {}
  onSignOut() {
    console.log("onSignOut called!");
  }
}
```
