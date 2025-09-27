# 4. Install and Configure Dependencies

For validation and documentation

```sh
npm install @nestjs/swagger class-validator class-transformer
```

```sh
npm install @nestjs/config
```

```sh
npm install @nestjs/mongoose mongoose
```

```sh
npm install class-validator class-transformer
```

**Update main.ts**

Add the swagger configuration and setup the global validation pipe at `main.ts`

<details>
<summary>src/main.ts</summary>

```TypeScript
import { NestFactory } from "@nestjs/core";
import { AppModule } from "./app.module";
import { DocumentBuilder, OpenAPIObject, SwaggerModule } from "@nestjs/swagger";
import { join } from "node:path";
import { writeFileSync } from "node:fs";
import { ValidationPipe } from "@nestjs/common";

async function bootstrap(): Promise<void> {
	const app = await NestFactory.create(AppModule, {
		bodyParser: false,
	});

	app.useGlobalPipes(
		new ValidationPipe({
			whitelist: true,
			forbidNonWhitelisted: true,
			transform: true, // <- here
			forbidUnknownValues: true,
			// disableErrorMessages: true,
		}),
	);

	const config = new DocumentBuilder()
		.setTitle("Application Title")
		.setDescription("The Application API Description")
		.setVersion("1.0")
		.addTag("app")
		.build();
	const documentFactory = (): OpenAPIObject => SwaggerModule.createDocument(app, config);

	// Setup SwaggerUI
	SwaggerModule.setup("api", app, documentFactory);

	// Save OpenAPI File
	const outputPath = join(process.cwd(), "swagger-spec.json");
	writeFileSync(outputPath, JSON.stringify(documentFactory(), null, 2));

	await app.listen(process.env.PORT ?? 3000);
}
bootstrap().catch((e): void => console.error(e));
```

</details>

**Add config module and mongoose module to app**

<details>
<summary>src/app.module.ts</summary>

```TypeScript
import { Module } from "@nestjs/common";
import { AppController } from "./app.controller";
import { AppService } from "./app.service";
import { ConfigModule, ConfigService } from "@nestjs/config";
import { MongooseModule, MongooseModuleFactoryOptions } from "@nestjs/mongoose";

@Module({
	imports: [
		ConfigModule.forRoot({
			isGlobal: true,
		}),
		// MongooseModule.forRoot("mongodb://root:example@localhost:27017/default_db"),
		MongooseModule.forRootAsync({
			imports: [ConfigModule],
			inject: [ConfigService],
			useFactory: (configService: ConfigService): MongooseModuleFactoryOptions => {
				const host = configService.get<string>("MONGO_HOST");
				const port = configService.get<string>("MONGO_PORT");
				const user = configService.get<string>("MONGO_INITDB_ROOT_USERNAME");
				const pass = configService.get<string>("MONGO_INITDB_ROOT_PASSWORD");
				const db = configService.get<string>("MONGO_INITDB_DATABASE");
				const uri = `mongodb://${user}:${pass}@${host}:${port}/${db}?authSource=admin`;
				// const uri = `mongodb://root:example@localhost:27017/default_db?authSource=admin`;

				return {
					uri,
				};
			},
		}),
	],
	controllers: [AppController],
	providers: [AppService],
})
export class AppModule {}

```

</details>

# 5. Set Up the Project

## Swagger (optional)

<details><summary>Settings</summary>
<dl><dd>

**Adjust `Swagger`**

> _A quick note:_
> Usually, in order to make the class properties visible to the `SwaggerModule`, you need to annotate them with the `@ApiProperty()` decorator [(docs)](https://docs.nestjs.com/openapi/types-and-parameters#types-and-parameters).

Alternatively, you can use the appropriate plugin, as described below. [(Swagger/CLI plugin)](https://docs.nestjs.com/openapi/cli-plugin) [(docs1)](https://www.prisma.io/blog/nestjs-prisma-relational-data-7D056s1kOabc#define-the-user-entity-and-dto-classes) [(docs2)](https://medium.com/@daiki01240/how-to-leverage-swagger-and-class-validator-in-nestjs-api-documentation-and-exporting-type-7577da98768d).

<!-- Before proceeding, make sure these packages are installed. -->

<!-- {
  "collection": "@nestjs/schematics",
  "sourceRoot": "src",
  "compilerOptions": {
    "plugins": ["@nestjs/swagger"]
  }
} -->

To [enable]() the plugin, open `nest-cli.json` and add the following plugins configuration. You can also use the options property to customize the behavior of the plugin.

<details><summary><strong>nest-cli.json</strong></summary>

```JavaScript
{
// ...
  "compilerOptions": {
	// ...
    "plugins": [
      {
        "name": "@nestjs/swagger",
        "options": {
          "classValidatorShim": true,
          "introspectComments": true,
          "skipAutoHttpCode": true
        }
      }
    ]
	// ...
  }
}
```

</details>

<br/>

The NestJS OpenAPI (Swagger) CLI plugin will automatically:

- Annotate DTO properties with `@ApiProperty` and set `required`, `type`, and `default`.
- Apply validation rules from `class-validator` (if `classValidatorShim` is enabled).
- Add response decorators to endpoints with proper status and types.
- Generate descriptions and examples from comments (if `introspectComments` is enabled).
- Generate and update Swagger (OpenAPI) documentation for your project.

</dd></dl>
</details>

# 6. Create and Set Up the Feature

    🏁 From now on, adding a new feature will involve pretty much the same set of steps. 🏁

### Generate a Feature (Module/Service/Controller/DTOs)

<details>
<summary><strong>Info</strong></summary>
<dl><dd>

You can generate the feature template with a single command and make a few adjustments:

```sh
nest generate resource notes
```

```sh
rm -rf ./src/notes/entities
```

```sh
echo "" > ./src/notes/dto/response-note.dto.ts
```

```sh
echo "" > ./src/notes/note.model.ts
```

```sh
echo "" > ./src/notes/dto/filter-note.dto.ts
```

Or you can create the files manually using separate commands.

```sh
nest generate module notes
```

```sh
nest generate service notes
```

```sh
nest generate controller notes
```

```sh
nest generate class notes/dto/create-note.dto --flat
```

```sh
nest generate class notes/dto/update-note.dto --flat
```

```sh
nest generate class notes/dto/response-note.dto --flat
```

```sh
echo "" > ./src/notes/note.model.ts
```

```sh
nest generate class notes/dto/filter-note.dto --flat
```

Note: Don’t add any content yet; just ensure the files are created.

- `src/notes/notes.service.ts` file
- `src/notes/notes.controller.ts` file
- `src/notes/notes.module.ts` file
- `src/notes/dto/create-note.dto.ts` file
- `src/notes/dto/update-note.dto.ts` file
- `src/notes/dto/response-note.dto.ts` file
- `src/notes/note.model.ts` file
- `src/notes/dto/filter-note.dto.ts` file

</dd></dl>
</details>

<br/>

### Create the model, schema, document

<details>
<summary>src/notes/note.model.ts</summary>

```TypeScript
import { Prop, Schema, SchemaFactory } from "@nestjs/mongoose";
import { HydratedDocument } from "mongoose";

@Schema({
	strict: true,
	timestamps: true,
})
export class Note {
	@Prop(String)
	public title!: string;

	@Prop({ required: false })
	public content!: string;

	@Prop({ required: false })
	public source!: string;

	@Prop({ type: Number, default: 0 })
	public priority!: number;

	@Prop({ type: Boolean })
	public isArchieved!: boolean;
}

export type NoteDocument = HydratedDocument<Note>;
export const NoteSchema = SchemaFactory.createForClass(Note);

```

</details>

### Register the MongooseModule `forFeature` in the `mports` Array of the Feature Module

<details>
<summary>src/notes/notes.module.ts</summary>

```TypeScript
import { Module } from "@nestjs/common";
import { NotesService } from "./notes.service";
import { NotesController } from "./notes.controller";
import { Note, NoteSchema } from "./note.model";
import { MongooseModule } from "@nestjs/mongoose";

@Module({
	imports: [MongooseModule.forFeature([{ name: Note.name, schema: NoteSchema }])],
	controllers: [NotesController],
	providers: [NotesService],
})
export class NotesModule {}
```

</details>

### Inject the Feature Model into the Feature Service

<details>
<summary>src/notes/notes.service.ts</summary>

```TypeScript
import { Injectable } from "@nestjs/common";
import { CreateNoteDto } from "./dto/create-note.dto";
import { UpdateNoteDto } from "./dto/update-note.dto";
import { InjectModel } from "@nestjs/mongoose";
import { Note, NoteDocument } from "./note.model";
import { Model } from "mongoose";

@Injectable()
export class NotesService {
	public constructor(@InjectModel(Note.name) private readonly noteModel: Model<NoteDocument>) {}

// ...
}
```

</details>

### Add Decorators to DTOs

<details>
<summary><strong>Info</strong></summary>
<dl><dd>

Add any decorators you need, then run `npm run format` and `npm run lint`, and fix any warnings or errors that may appear.”

The result should include the following files:

<details><summary><strong>src/notes/dto/create-note.dto.ts</strong></summary>

```ts
// import { ApiProperty } from "@nestjs/swagger";
import {
  IsNotEmpty,
  IsNumber,
  IsOptional,
  IsString,
  Length,
  Max,
  MaxLength,
  Min,
  MinLength,
} from "class-validator";

export class CreateNoteDto {
  // @ApiProperty()
  @IsNotEmpty()
  @IsString()
  @MinLength(3)
  @MaxLength(20)
  public title!: string;

  // @ApiProperty()
  @IsOptional()
  @IsString()
  @Length(5, 50)
  public content?: string;

  // @ApiProperty()
  @IsOptional()
  @IsString()
  @Length(1, 30)
  public source?: string;

  // @ApiProperty()
  @IsOptional()
  @IsNumber()
  // @IsPositive()
  @Min(0)
  @Max(5)
  public priority?: number;

  public constructor(data: CreateNoteDto) {
    if (data?.title != null) this.title = data?.title;
    if (data?.content != null) this.content = data?.content;
    if (data?.source != null) this.source = data?.source;
    if (data?.priority != null) this.priority = data?.priority;
  }
}
```

</details>

<br/>

<details>
	<summary><strong>src/notes/dto/update-note.dto.ts</strong></summary>

```ts
import {
  IsBoolean,
  IsNotEmpty,
  IsNumber,
  IsOptional,
  IsString,
  Length,
  Max,
  MaxLength,
  Min,
  MinLength,
} from "class-validator";

export class UpdateNoteDto {
  @IsOptional()
  @IsNotEmpty()
  @IsString()
  @MinLength(3)
  @MaxLength(20)
  public title?: string;

  @IsOptional()
  @IsString()
  @Length(5, 50)
  public content?: string;

  @IsOptional()
  @IsString()
  @Length(1, 30)
  public source?: string;

  @IsOptional()
  @IsNumber()
  // @IsPositive()
  @Min(0)
  @Max(5)
  public priority?: number;

  @IsOptional()
  @IsBoolean()
  public isArchived?: boolean;

  public constructor(data: UpdateNoteDto) {
    if (data?.title != null) this.title = data?.title;
    if (data?.content != null) this.content = data?.content;
    if (data?.source != null) this.source = data?.source;
    if (data?.priority != null) this.priority = data?.priority;
    if (data?.isArchived != null) this.isArchived = data?.isArchived;
  }
}
```

</details>

<br/>

<details><summary><strong>src/notes/dto/response-note.dto.ts</strong></summary>

```ts
export class ResponseNoteDto {
  public id!: string;
  public title!: string;
  public content?: string | null;

  public constructor(data: ResponseNoteDto) {
    if (data?.id != null) this.id = data?.id;
    if (data?.title != null) this.title = data?.title;
    if (data?.content != null) this.content = data?.content;
  }
}
```

<!-- ```ts
// src/notes/dto/login-note.dto.ts
import { ApiProperty } from "@nestjs/swagger";
import { IsEmail, IsString, MinLength } from "class-validator";

export class LoginUserDto {
	@ApiProperty({
		description: "User email address",
		example: "user@example.com",
	})
	@IsEmail()
	public email: string;

	@ApiProperty({
		description: "User password",
		example: "strongPassword123",
		minLength: 6,
	})
	@IsString()
	@MinLength(6)
	public password: string;

	public constructor(data: LoginUserDto) {
		this.email = data.email;
		this.password = data.password;
	}
}

``` -->

</details>

<details><summary><strong>src/notes/dto/filter-note.dto.ts</strong></summary>

```ts
import { IsNumber, IsOptional } from "class-validator";

export class FilterNoteDto {
  // @IsString()
  // @IsOptional()
  // public category?: string;

  @IsNumber()
  @IsOptional()
  public limit?: number;
}
```

</details>

</dd></dl>
</details>

<br/>

### Add `ValidationPipe` (+Transformation)

<details>
<summary><strong>Info</strong></summary>
<dl><dd>

<!-- **Update the Controller and Service, Add Pipes** -->

#### **Validation and Transformation**

 <!-- (and others `сlass-validator` pipes)  -->

Validation and transformation can be enabled in the following ways:

<details>
<summary>Globally: In `src/main.ts`, apply the `ValidationPipe` to the entire application</summary>

```TypeScript
import { NestFactory } from "@nestjs/core";
import { AppModule } from "./app.module";
import { ValidationPipe } from "@nestjs/common"; // <- here

async function bootstrap(): Promise<void> {
  const app = await NestFactory.create(AppModule, {
    bodyParser: false,
  });

  /* here */
  app.useGlobalPipes(
    new ValidationPipe({
      whitelist: true,
      transform: true,
      forbidNonWhitelisted: true,
      forbidUnknownValues: true,
      // disableErrorMessages: true,
    })
  );

  await app.listen(process.env.PORT ?? 3000);
}

bootstrap().catch((e): void => console.error(e));
```

</details>

<details>
<summary>At the controller level: apply `ValidationPipe` using the `@UsePipes` decorator.</summary>

Use the `@UsePipes` to apply the `ValidationPipe` to an entire controller

```TypeScript
@UsePipes(
	new ValidationPipe({
		whitelist: true,
		transform: true,
		forbidNonWhitelisted: true,
		forbidUnknownValues: true,
		// disableErrorMessages: true,
	})
)
@Controller("notes")
export class NotesController {}
```

</details>

<details>
<summary>At the method level: apply `ValidationPipe` using the `@UsePipes` decorator.</summary>

```TypeScript
@Post()
@UsePipes(
	new ValidationPipe({
		whitelist: true,
		transform: true,
		forbidNonWhitelisted: true,
		forbidUnknownValues: true,
		// disableErrorMessages: true,
	})
)
async create(/* ... */) {}
```

</details>

<details>
<summary>At the parameter level: use `ParseIntPipe` or `ParseBoolPipe` to cast values explicitly</summary>

<br/>
You can explicitly cast values using the `ParseIntPipe` or `ParseBoolPipe` inside `@Param()` or `@Query()` decorators. A separate pipe is used for each, so `ValidationPipe` is not needed for them to work:

```ts

@Get(':id')
findOne(
  @Param('id', ParseIntPipe) id: number,
  @Query('sort', ParseBoolPipe) sort: boolean,
) {
  console.log(typeof id === 'number'); // true
  console.log(typeof sort === 'boolean'); // true
  return 'This action returns a user';
}
```

</details>

<!-- </details> -->

<!-- <details>
<summary><strong>**How to use `class-validator` validation pipes**</strong></summary> -->

<br/>

> **How do pipes work?**
> If a ValidationPipe is enabled (globally, at the controller, or method level), it automatically validates the DTO passed as an argument.

```TypeScript
create(@Body() createUserDto: CreateUserDto) {}
```

<br/>

> With the auto-transformation option enabled, the ValidationPipe will also perform primitive type conversion. For example, the findOne() method below takes an id path parameter and automatically converts its type to a number.

```TypeScript
@Get(':id')
findOne(@Param('id') id: number) {
  console.log(typeof id === 'number'); // true
  return 'This action returns a user';
}
```

<br/>

📝 If you need to validate arrays in NestJS, refer to [the official documentation](https://docs.nestjs.com/techniques/validation#parsing-and-validating-arrays).

<!-- Now implement the desired logic using the full power of `class-validator`, `@nestjs/swagger`, `better-auth`, and more. -->

**Docs:**

- [(NestJS validation pipes)](https://docs.nestjs.com/techniques/validation)
- [(Better Auth decorators)](https://github.com/ThallesP/nestjs-better-auth)
- [(OpenAPI/Swagger general decorators)](https://docs.nestjs.com/openapi/decorators)
- [(OpenAPI/Swagger response decorators)](https://docs.nestjs.com/openapi/operations#responses)
- [(docs)](https://docs.nestjs.com/techniques/validation#validation)
- [(more docs)](https://docs.nestjs.com/pipes#class-validator)
</details>

<br/>

# Example resulting files are listed below:

<details>
<summary>src/notes/notes.service.ts</summary>

```TypeScript
import { Injectable, NotFoundException } from "@nestjs/common";
import { InjectModel } from "@nestjs/mongoose";
import { Note, NoteDocument } from "./note.model";
import { DeleteResult, Model, Types } from "mongoose";
import { FilterNoteDto } from "./dto/filter-note.dto";
import { CreateNoteDto } from "./dto/create-note.dto";
import { UpdateNoteDto } from "./dto/update-note.dto";
// import { FilterNoteDto } from "./dto/filter-note.dto";

@Injectable()
export class NoteService {
	public constructor(@InjectModel(Note.name) private readonly noteModel: Model<NoteDocument>) {}

	public async create(dto: CreateNoteDto): Promise<NoteDocument> {
		// const instance2 = await this.noteModel.create([dto], { validateBeforeSave: true });
		const noteInstance = new this.noteModel(dto);
		await noteInstance.save({
			validateBeforeSave: true,
		});

		return noteInstance;
	}

	public async update(id: string, dto: UpdateNoteDto): Promise<NoteDocument> {
		const updatedNote = await this.noteModel.findByIdAndUpdate(
			new Types.ObjectId(id),
			{ $set: dto },
			{ new: true, runValidators: true }
		);

		if (!updatedNote) {
			throw new NotFoundException(`Note with ID ${id} not found`);
		}

		return updatedNote;
	}

	public async delete(id: string): Promise<NoteDocument | null> {
		const deletedNote = await this.noteModel.findByIdAndDelete(new Types.ObjectId(id));
		if (!deletedNote) {
			throw new NotFoundException(`Note with ID ${id} not found`);
		}
		return deletedNote;
	}

	public async getById(id: string): Promise<NoteDocument | null> {
		const note = await this.noteModel.findById(new Types.ObjectId(id));
		if (!note) {
			throw new NotFoundException(`Note with ID ${id} not found`);
		}
		return note;
	}

	public async getByProduct(productId: string): Promise<NoteDocument[] | null> {
		return this.noteModel
			.find({ productId: new Types.ObjectId(productId) })
			.lean()
			.exec();
	}

	public async getAll(filter: FilterNoteDto): Promise<NoteDocument[]> {
		return this.noteModel
			.find(filter)
			.limit(filter.limit ?? 10)
			.exec();
	}

	public async deleteByProductId(productId: string): Promise<DeleteResult> {
		return this.noteModel.deleteMany({ productId: new Types.ObjectId(productId) }).exec();
	}
}

```

</details>

<br/>

<details>
<summary>src/note/notes.controller.ts</summary>

```TypeScript
import { Injectable, NotFoundException } from "@nestjs/common";
import { InjectModel } from "@nestjs/mongoose";
import { Note, NoteDocument } from "./note.model";
import { Model, Types } from "mongoose";
import { FilterNoteDto } from "./dto/filter-note.dto";
import { CreateNoteDto } from "./dto/create-note.dto";
import { UpdateNoteDto } from "./dto/update-note.dto";
// import { FilterNoteDto } from "./dto/filter-note.dto";

@Injectable()
export class NoteService {
	public constructor(@InjectModel(Note.name) private readonly noteModel: Model<NoteDocument>) {}

	public async create(dto: CreateNoteDto): Promise<NoteDocument> {
		// const instance2 = await this.noteModel.create([dto], { validateBeforeSave: true });
		const noteInstance = new this.noteModel(dto);
		await noteInstance.save({
			validateBeforeSave: true,
		});

		return noteInstance;
	}

	public async update(id: string, dto: UpdateNoteDto): Promise<NoteDocument> {
		const updatedNote = await this.noteModel.findByIdAndUpdate(
			new Types.ObjectId(id),
			{ $set: dto },
			{ new: true, runValidators: true }
		);

		if (!updatedNote) {
			throw new NotFoundException(`Note with ID ${id} not found`);
		}

		return updatedNote;
	}

	public async delete(id: string): Promise<NoteDocument | null> {
		const deletedNote = await this.noteModel.findByIdAndDelete(new Types.ObjectId(id));
		if (!deletedNote) {
			throw new NotFoundException(`Note with ID ${id} not found`);
		}
		return deletedNote;
	}

	public async getById(id: string): Promise<NoteDocument | null> {
		const note = await this.noteModel.findById(new Types.ObjectId(id));
		if (!note) {
			throw new NotFoundException(`Note with ID ${id} not found`);
		}
		return note;
	}

	public async getAll(filter: FilterNoteDto): Promise<NoteDocument[]> {
		return this.noteModel
			.find(filter)
			.limit(filter.limit ?? 10)
			.exec();
	}
}
```

</details>

<details>
<summary>src/notes/notes.module.ts</summary>

```ts
import { Module } from "@nestjs/common";
import { NoteService } from "./notes.service";
import { NoteController } from "./notes.controller";
import { Note, NoteSchema } from "./note.model";
import { MongooseModule } from "@nestjs/mongoose";

@Module({
  imports: [
    MongooseModule.forFeature([{ name: Note.name, schema: NoteSchema }]),
  ],
  controllers: [NoteController],
  providers: [NoteService],
})
export class NotesModule {}
```

</details>
