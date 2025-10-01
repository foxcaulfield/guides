Perform commont settings

---

<style> 
.hdr1 {
	font-size: 35px;
	text-align:center;
	padding-top: 40px;
}

/* .hdr2{
	font-size: 27.5px;
	text-align:center;
	padding-top: 30px;
} */

.hdr3{
	font-size: 20px;
	text-align:center;
	padding-top: 20px;
}

.code-block-header {
	padding-bottom: 0px;
  	font-family: monospace;
	font-weight: bold;
	color: #a84923ff;
	font-size: 1.05em;
}
</style>

# 4. <div class="hdr1">4. Install and Configure Dependencies</div>

### <div class="hdr3">Config and dotenv</div>

- [(docs)](https://www.npmjs.com/package/dotenv)

Install:

```sh
npm install dotenv --save
```

```sh
npm install @nestjs/config
```

Create and configure a `.env` file:

```sh
echo "" > .env
```

Add config module to the imports array of your root app module:

<div class="code-block-header">src/app.module.ts</div>

```ts
import { Module } from "@nestjs/common";
import { ConfigModule } from "@nestjs/config";

@Module({
  imports: [
    // ...
    ConfigModule.forRoot({
      isGlobal: true,
      envFilePath: "./.env",
    }),
    // ...
  ],
  // ...
})
export class AppModule {}
```

### <div class="hdr3">Docker + MongoDB</div>

Update the `.env` file

`.env`

```env
MONGO_INITDB_ROOT_USERNAME=root
MONGO_INITDB_ROOT_PASSWORD=example
MONGO_INITDB_DATABASE=default_db

MONGO_HOST=0.0.0.0
MONGO_PORT=27017
```

Create a `docker-compose.yaml` file:

```sh
echo "" > docker-compose.yaml
```

Add the following configuration:

<div class="code-block-header">docker-compose.yaml</div>

```yaml
services:
  mongo:
    image: mongo:8-noble
    restart: always
    container_name: mongo_container
    env_file: ./.env
    ports:
      - 27017:27017
    volumes:
      - ./mongo-data:/data/db
    command: --wiredTigerCacheSizeGB 1.5
    # networks:
    #     - backend-network

  mongo-express:
    image: mongo-express
    restart: always
    ports:
      - 8081:8081
    env_file: ./.env
    depends_on:
      - mongo

    environment:
      ME_CONFIG_MONGODB_URL: mongodb://root:example@mongo:27017/
      ME_CONFIG_BASICAUTH_ENABLED: true
      ME_CONFIG_BASICAUTH_USERNAME: root
      ME_CONFIG_BASICAUTH_PASSWORD: example
# networks:
#     backend-network:
```

Then start the container:

```sh
docker compose up -d
```

Or start only the `db` service:

```sh
docker compose up db -d
```

### <div class="hdr3">Mongoose</div>

```sh
npm install @nestjs/mongoose mongoose
```

**Add config module and mongoose module to app**

<div class="code-block-header">src/app.module.ts</div>

```TypeScript
import { Module } from "@nestjs/common";
import { ConfigModule, ConfigService } from "@nestjs/config";
import { MongooseModule, MongooseModuleFactoryOptions } from "@nestjs/mongoose";

// MongooseModule.forRoot("mongodb://root:example@localhost:27017/default_db"),
// const uri = `mongodb://root:example@localhost:27017/default_db?authSource=admin`;

@Module({
	imports: [
	// ...
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
				return {
					uri,
				};
			},
		}),
	// ...
	],
// ...
})
export class AppModule {}

```

### <div class="hdr3">Validation</div>

For validation and documentation purposes

```sh
npm install @nestjs/swagger class-validator class-transformer
```

Add the swagger configuration and setup the global validation pipe at `main.ts`

<div class="code-block-header">src/main.ts</div>

```TypeScript
import { NestFactory } from "@nestjs/core";
import { AppModule } from "./app.module";
import { DocumentBuilder, OpenAPIObject, SwaggerModule } from "@nestjs/swagger";
import { join } from "node:path";
import { writeFileSync } from "node:fs";
import { ValidationPipe } from "@nestjs/common";

async function bootstrap(): Promise<void> {
	const app = await NestFactory.create(AppModule, {
		// bodyParser: false,
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
	const documentFactory = (): OpenAPIObject =>
			SwaggerModule.createDocument(app, config);

	// Setup SwaggerUI
	SwaggerModule.setup("api", app, documentFactory);

	// Save OpenAPI File
	const outputPath = join(process.cwd(), "swagger-spec.json");
	writeFileSync(outputPath, JSON.stringify(documentFactory(), null, 2));

	await app.listen(process.env.PORT ?? 3000);
}
bootstrap().catch((e): void => console.error(e));
```

### <div class="hdr3">Swagger auto annotations (optional)</div>

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

nest-cli.json

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

The NestJS OpenAPI (Swagger) CLI plugin will automatically:

- Annotate DTO properties with `@ApiProperty` and set `required`, `type`, and `default`.
- Apply validation rules from `class-validator` (if `classValidatorShim` is enabled).
- Add response decorators to endpoints with proper status and types.
- Generate descriptions and examples from comments (if `introspectComments` is enabled).
- Generate and update Swagger (OpenAPI) documentation for your project.

# <div class="hdr1">5. Set Up the Project</div>

<!-- Create and Set Up the Feature -->

<!-- ### <div class="hdr3">Create and Set Up the Feature</div> -->

    🏁 From now on, adding a new feature
    will involve pretty much the same set of steps. 🏁

### <div class="hdr3">Generate a Feature files (Module/Service/Controller/DTOs)</div>

You can generate the feature template with a single command and make a few adjustments:

```sh
while true; do echo y; sleep 1; done | nest generate resource rooms &&
rm -rf ./src/rooms/entities &&
nest generate class rooms/dto/response-room.dto --flat &&
nest generate class rooms/dto/filter-room.dto --flat &&
echo "" > ./src/rooms/room.model.ts
```

Or you can create the files manually using separate commands.

```sh
nest generate module rooms &&
nest generate service rooms &&
nest generate controller rooms &&
nest generate class rooms/dto/create-room.dto --flat &&
nest generate class rooms/dto/update-room.dto --flat &&
nest generate class rooms/dto/response-room.dto --flat &&
nest generate class rooms/dto/filter-room.dto --flat &&
echo "" > ./src/rooms/room.model.ts &&
```

Note: Don’t add any content yet; just ensure the files are created.

- `src/rooms/rooms.service.ts` file
- `src/rooms/rooms.controller.ts` file
- `src/rooms/rooms.module.ts` file
- `src/rooms/dto/create-room.dto.ts` file
- `src/rooms/dto/update-room.dto.ts` file
- `src/rooms/dto/response-room.dto.ts` file
- `src/rooms/dto/filter-room.dto.ts` file
- `src/rooms/room.model.ts` file

**Generate reservations feature**

> 📋 Do the same set of steps for the `reservation` feature

### <div class="hdr3">Provide a model, a schema, a document, DTOs</div>

<details>
<summary><span class="code-block-header">src/rooms/room.model.ts</span>
</summary>

```TypeScript
import { Prop, Schema, SchemaFactory } from "@nestjs/mongoose";
import { HydratedDocument, Model } from "mongoose";

export enum RoomType {
	STANDARD_ROOM = "STANDARD_ROOM",
	DELUXE_ROOM = "DELUXE_ROOM",
	STUDIO_ROOM = "STUDIO_ROOM",
	SUITE = "SUITE",
	FAMILY_ROOM = "FAMILY_ROOM",
}

export enum RoomStatus {
	AVAILABLE = "AVAILABLE",
	OCCUPIED = "OCCUPIED",
	MAINTENANCE = "MAINTENANCE",
	CLEANING = "CLEANING",
}

// // Nested schema
// @Schema({ _id: false })
// export class Amenity {
// 	@Prop({ required: true, trim: true })
// 	public name!: string;

// 	@Prop({ default: false })
// 	public isPremium?: boolean;

// 	@Prop()
// 	public description?: string;
// }

// Main schema
@Schema({
	strict: true,
	timestamps: true,
	toJSON: {
		virtuals: false,
	},
})
export class Room {
	@Prop({
		type: Number,
		unique: true,
		required: true,
		min: 1,
		max: 1000,
		index: true,
	})
	public roomNumber!: number;

	@Prop({
		type: String,
		enum: RoomType,
		default: RoomType.STANDARD_ROOM,
	})
	public roomType!: RoomType;

	@Prop({
		type: String,
		enum: RoomStatus,
		default: RoomStatus.AVAILABLE,
	})
	public status!: RoomStatus;

	@Prop(Boolean)
	public hasSeaView!: boolean;

	// @Prop({
	// 	type: Number,
	// 	min: 1,
	// 	max: 6,
	// })
	// public maxOccupancy!: number;

	// @Prop({
	// 	type: [Amenity],
	// 	validate: {
	// 		validator: (arr: Amenity[]): boolean => arr.length <= 5,
	// 		message: "Amenity limit is 5",
	// 	},
	// })
	// public amenities!: Amenity[];

	// @Prop({ type: Date })
	// public lastMaintenance?: Date;

	// /* Virtual fields */
	// public get displayName(): string {
	// 	return `Room ${this.roomNumber} (${this.roomType})`;
	// }
	// public get isAvailable(): boolean {
	// 	return this.status === RoomStatus.AVAILABLE;
	// }

	// /* Methods */
	// public hasAmenity(amenityName: string): boolean {
	// 	return this.amenities?.some((a): boolean => a.name === amenityName) ?? false;
	// }

	// /* Static */
	// public static async findByType(this: RoomModelType, roomType: RoomType): Promise<RoomDocument[]> {
	// 	return this.find({ roomType }).exec();
	// }
}
export type RoomDocument = HydratedDocument<Room>;
export type RoomModelType = Model<RoomDocument> & typeof Room;
export const RoomSchema = SchemaFactory.createForClass(Room);

```

</details>

<details>
<summary><span class="code-block-header">src/rooms/dto/create-room.dto.ts</span></summary>

```ts
import {
  IsEnum,
  IsInt,
  IsBoolean,
  IsArray,
  ValidateNested,
  Min,
  Max,
  IsOptional,
  IsDate,
  IsNumber,
} from "class-validator";
import { Type } from "class-transformer";
import { RoomStatus, RoomType } from "../room.model";

// export class AmenityDto {
//   @IsBoolean()
//   @IsOptional()
//   public isPremium?: boolean;

//   @IsOptional()
//   public description?: string;
// }

export class CreateRoomDto {
  @IsInt()
  @Min(1)
  @Max(1000)
  public roomNumber!: number;

  @IsEnum(RoomType)
  public roomType!: RoomType;

  @IsEnum(RoomStatus)
  @IsOptional()
  public status?: RoomStatus;

  @IsBoolean()
  @IsOptional()
  public hasSeaView?: boolean;

  //   @IsInt()
  //   @Min(1)
  //   @Max(6)
  //   public maxOccupancy!: number;

  //   @IsArray()
  //   @ValidateNested({ each: true })
  //   @Type((): typeof AmenityDto => AmenityDto)
  //   @IsOptional()
  //   public amenities?: AmenityDto[];

  //   @IsDate()
  //   @Type((): typeof Date => Date)
  //   @IsOptional()
  //   public lastMaintenance?: Date;
}
```

</details>

<details>
<summary><span class="code-block-header">src/rooms/dto/update-room.dto.ts</span></summary>

```ts
import { PartialType } from "@nestjs/mapped-types";
import { CreateRoomDto } from "./create-room.dto";

export class UpdateRoomDto extends PartialType(CreateRoomDto) {}
```

</details>

<details>
<summary><span class="code-block-header">src/rooms/dto/response-room.dto.ts</span></summary>

```ts
import { Exclude, Expose, Transform } from "class-transformer";
import { RoomStatus, RoomType } from "../room.model";

// export class AmenityResponseDto {
//   @Expose()
//   public name!: string;

//   @Expose()
//   public isPremium?: boolean;

//   @Expose()
//   public description?: string;
// }

export class RoomResponseDto {
  @Expose()
  public id!: string;

  @Expose()
  public roomNumber!: number;

  @Expose()
  public roomType!: RoomType;

  @Expose()
  public status!: RoomStatus;

  @Expose()
  public hasSeaView!: boolean;

  //   @Expose()
  //   public maxOccupancy!: number;

  //   @Expose()
  //   @Transform((params): AmenityResponseDto[] => params.value || [])
  //   public amenities!: AmenityResponseDto[];

  //   @Expose()
  //   public lastMaintenance?: Date;

  @Expose()
  public createdAt!: Date;

  @Expose()
  public updatedAt!: Date;

  // @Expose()
  // public get displayName(): string {
  // 	return `Room ${this.roomNumber} (${this.roomType})`;
  // }

  // @Expose()
  // public get isAvailable(): boolean {
  // 	return this.status === RoomStatus.AVAILABLE;
  // }
}
```

</details>

<details>
<summary><span class="code-block-header">src/rooms/dto/filter-room.dto.ts</span></summary>

```ts
import {
  IsEnum,
  IsInt,
  IsBoolean,
  IsOptional,
  Min,
  Max,
} from "class-validator";
import { Type } from "class-transformer";
import { RoomType, RoomStatus } from "../room.model";

export class RoomFilterDto {
  @IsEnum(RoomType)
  @IsOptional()
  public roomType?: RoomType;

  @IsEnum(RoomStatus)
  @IsOptional()
  public status?: RoomStatus;

  @IsBoolean()
  @IsOptional()
  @Type((): typeof Boolean => Boolean)
  public hasSeaView?: boolean;

  //   @IsInt()
  //   @Min(1)
  //   @Max(6)
  //   @IsOptional()
  //   @Type((): typeof Number => Number)
  //   public minOccupancy?: number;

  //   @IsInt()
  //   @Min(0)
  //   @IsOptional()
  //   @Type((): typeof Number => Number)
  //   public maxPrice?: number;

  @IsInt()
  @Min(1)
  @IsOptional()
  @Type((): typeof Number => Number)
  public page?: number = 1;

  @IsInt()
  @Min(1)
  @Max(100)
  @IsOptional()
  @Type((): typeof Number => Number)
  public limit?: number = 10;
}
```

</details>

### <div class="hdr3">Provide a module, a service, a controller</div>

#### Register the MongooseModule `forFeature` in the `imports` Array of the Feature Module

<details>
<summary><span class="code-block-header">src/rooms/rooms.module.ts</span></summary>

```TypeScript
import { Module } from "@nestjs/common";
import { RoomsService } from "./rooms.service";
import { RoomsController } from "./rooms.controller";
import { MongooseModule } from "@nestjs/mongoose";
import { Room, RoomSchema } from "./room.model";

@Module({
	imports: [
		MongooseModule.forFeature([
			{
				name: Room.name,
				schema: RoomSchema,
			},
		]),
	],
	providers: [RoomsService],
	controllers: [RoomsController],
})
export class RoomsModule {}

```

</details>

#### Inject the Feature Model into the Feature Service and Provide the Rest

<details>
<summary><span class="code-block-header">src/rooms/rooms.service.ts</span></summary>

```TypeScript
import { ConflictException, Injectable, NotFoundException } from "@nestjs/common";
import { InjectModel } from "@nestjs/mongoose";
import { RootFilterQuery, Types } from "mongoose";
import { Room, RoomDocument, RoomStatus, RoomType, type RoomModelType } from "./room.model";
import { CreateRoomDto } from "./dto/create-room.dto";
import { UpdateRoomDto } from "./dto/update-room.dto";
import { RoomFilterDto } from "./dto/filter-room.dto";
import { ResponseRoomDto } from "./dto/response-room.dto";
import { plainToInstance } from "class-transformer";

@Injectable()
export class RoomsService {
	public constructor(
		@InjectModel(Room.name)
		private readonly roomModel: RoomModelType
	) {}

	public async create(dto: CreateRoomDto): Promise<ResponseRoomDto> {
		const existingRoom = await this.roomModel.findOne({
			roomNumber: dto.roomNumber,
		});

		if (existingRoom) {
			throw new ConflictException(`Room with number ${dto.roomNumber} already exists`);
		}

		const roomInstance = new this.roomModel(dto);
		await roomInstance.save({
			validateBeforeSave: true,
		});


		const responseDto = plainToInstance(
			ResponseRoomDto,
			roomInstance,
			{
				excludeExtraneousValues: true
			});


		return responseDto;
	}

	public async findAll(filterDto: RoomFilterDto): Promise<{
		rooms: RoomDocument[];
		total: number;
		page: number;
		limit: number;
		pages: number;
	}> {
		const { page, limit, ...filters } = filterDto;
		const skip = page === undefined || limit === undefined ? 0 : (page - 1) * limit;
		const limitValue = limit ?? 10;
		const pageValue = page ?? 0;
		const query = this.buildFilterQuery(filters);

		const [rooms, total] = await Promise.all([
			this.roomModel.find(query).sort({ roomNumber: 1 }).skip(skip).limit(limitValue).exec(),
			this.roomModel.countDocuments(query).exec(),
		]);

		return {
			rooms,
			total,
			page: pageValue,
			limit: limitValue,
			pages: Math.ceil(total / limitValue),
		};
	}

	public async findById(id: string): Promise<RoomDocument> {
		const room = await this.roomModel.findById(new Types.ObjectId(id));

		if (!room) {
			throw new NotFoundException(`Room with ID ${id} not found`);
		}

		return room;
	}

	public async update(id: string, dto: UpdateRoomDto): Promise<RoomDocument> {
		/* if (dto.roomNumber) {
			const existingRoom = await this.roomModel.findOne({
				roomNumber: dto.roomNumber,
				_id: { $ne: new Types.ObjectId(id) },
			});

			if (existingRoom) {
				throw new ConflictException(`Room with number ${dto.roomNumber} already exists`);
			}
		} */

		const updatedRoom = await this.roomModel.findByIdAndUpdate(
			new Types.ObjectId(id),
			{ $set: dto },
			{ new: true, runValidators: true },
		);

		if (!updatedRoom) {
			throw new NotFoundException(`Room with ID ${id} not found`);
		}

		return updatedRoom;
	}

	public async remove(id: string): Promise<void> {
		const result = await this.roomModel.findByIdAndDelete(new Types.ObjectId(id));

		if (!result) {
			throw new NotFoundException(`Room with ID ${id} not found`);
		}
	}

	public async findByType(roomType: RoomType): Promise<RoomDocument[]> {
		return this.roomModel.findByType(roomType);
	}

	public async findAvailableRooms(): Promise<RoomDocument[]> {
		return this.roomModel
			.find({
				status: RoomStatus.AVAILABLE,
			})
			.sort({ roomNumber: 1 })
			.exec();
	}

	private buildFilterQuery(filters: Partial<RoomFilterDto>): RootFilterQuery<RoomDocument> {
		const query: RootFilterQuery<RoomDocument> = {};

		if (filters.roomType) {
			query.roomType = filters.roomType;
		}

		if (filters.status) {
			query.status = filters.status;
		}

		if (filters.hasSeaView !== undefined) {
			query.hasSeaView = filters.hasSeaView;
		}

		if (filters.minOccupancy) {
			query.maxOccupancy = { $gte: filters.minOccupancy };
		}

		if (filters.maxPrice) {
			query["pricing.basePrice"] = { $lte: filters.maxPrice };
		}

		return query;
	}

	// public async update(id: string, dto: UpdateRoomDto): Promise<RoomDocument> {
	// 	const updatedRoom = await this.roomModel.findByIdAndUpdate(
	// 		new Types.ObjectId(id),
	// 		{ $set: dto },
	// 		{ new: true, runValidators: true },
	// 	);

	// 	if (!updatedRoom) {
	// 		throw new NotFoundException(`Room with ID ${id} not found`);
	// 	}

	// 	return updatedRoom;
	// }

	// public async delete(id: string): Promise<RoomDocument | null> {
	// 	const deletedRoom = await this.roomModel.findByIdAndDelete(new Types.ObjectId(id));
	// 	if (!deletedRoom) {
	// 		throw new NotFoundException(`Room with ID ${id} not found`);
	// 	}
	// 	return deletedRoom;
	// }

	// public async getById(id: string): Promise<RoomDocument | null> {
	// 	const room = await this.roomModel.findById(new Types.ObjectId(id));
	// 	if (!room) {
	// 		throw new NotFoundException(`Room with ID ${id} not found`);
	// 	}
	// 	return room;
	// }

	// public async getAll(filter: { limit?: number }): Promise<RoomDocument[]> {
	// 	return this.roomModel
	// 		.find(filter)
	// 		.limit(filter.limit ?? 10)
	// 		.exec();
	// }
}

```

</details>

#### Setup the Controller

<details>
<summary><span class="code-block-header">src/rooms/rooms.controller.ts</span></summary>

```ts
import {
  Controller,
  Get,
  Post,
  Body,
  Patch,
  Param,
  Delete,
  Query,
  UseInterceptors,
  // ParseIntPipe,
  HttpStatus,
  HttpCode,
  ClassSerializerInterceptor,
} from "@nestjs/common";
// import { RoomService } from "./room.service";
import { CreateRoomDto } from "./dto/create-room.dto";
import { UpdateRoomDto } from "./dto/update-room.dto";
import { RoomResponseDto } from "./dto/response-room.dto";
import { RoomsService } from "./rooms.service";
import { RoomDocument, RoomType } from "./room.model";
import { RoomFilterDto } from "./dto/filter-room.dto";
// import { RoomFilterDto } from "./dto/room-filter.dto";
// import { RoomResponseDto } from "./dto/room-response.dto";
// import { SerializeInterceptor } from "../interceptors/serialize.interceptor";
import { ResponseRoomDto } from "./dto/response-room.dto";

@Controller("rooms")
@UseInterceptors(new ClassSerializerInterceptor(RoomResponseDto))
export class RoomController {
  public constructor(private readonly roomService: RoomsService) {}

  @Post()
  public async create(
    @Body() createRoomDto: CreateRoomDto
  ): Promise<ResponseRoomDto> {
    return await this.roomService.create(createRoomDto);
  }

  @Get()
  public async findAll(@Query() filterDto: RoomFilterDto): Promise<{
    rooms: RoomDocument[];
    total: number;
    page: number;
    limit: number;
    pages: number;
  }> {
    return await this.roomService.findAll(filterDto);
  }

  @Get("available")
  public async findAvailable(): Promise<RoomDocument[]> {
    return await this.roomService.findAvailableRooms();
  }

  @Get("type/:type")
  public async findByType(
    @Param("type") type: RoomType
  ): Promise<RoomDocument[]> {
    return await this.roomService.findByType(type);
  }

  @Get(":id")
  public async findOne(@Param("id") id: string): Promise<RoomDocument> {
    return await this.roomService.findById(id);
  }

  @Patch(":id")
  public async update(
    @Param("id") id: string,
    @Body() updateRoomDto: UpdateRoomDto
  ): Promise<RoomDocument> {
    return await this.roomService.update(id, updateRoomDto);
  }

  @Delete(":id")
  @HttpCode(HttpStatus.NO_CONTENT)
  public async remove(@Param("id") id: string): Promise<void> {
    await this.roomService.remove(id);
  }
}
```

</details>

### <div class="hdr3">Add `ValidationPipe` (+Transformation)</div>

<!-- Info -->

<!-- **Update the Controller and Service, Add Pipes** -->

#### **Validation and Transformation**

 <!-- (and others `сlass-validator` pipes)  -->

<details>
<summary>Validation and transformation can be enabled in the following ways:
</summary>

Globally: In `src/main.ts`, apply the `ValidationPipe` to the entire application

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

At the controller level: apply `ValidationPipe` using the `@UsePipes` decorator.

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

At the method level: apply `ValidationPipe` using the `@UsePipes` decorator.

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

At the parameter level: use `ParseIntPipe` or `ParseBoolPipe` to cast values explicitly

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

<!-- </details> -->

<!-- <details>
<summary><strong>**How to use `class-validator` validation pipes**</strong></summary> -->

> **How do pipes work?**
> If a ValidationPipe is enabled (globally, at the controller, or method level), it automatically validates the DTO passed as an argument.

```TypeScript
create(@Body() createUserDto: CreateUserDto) {}
```

> With the auto-transformation option enabled, the ValidationPipe will also perform primitive type conversion. For example, the findOne() method below takes an id path parameter and automatically converts its type to a number.

```TypeScript
@Get(':id')
findOne(@Param('id') id: number) {
  console.log(typeof id === 'number'); // true
  return 'This action returns a user';
}
```

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

# <div class="hdr1">Example resulting files are listed below</div>

<details>
<summary><div class="code-block-header">src/reservation.model.ts</div></summary>

```ts
import { Prop, Schema, SchemaFactory } from "@nestjs/mongoose";
import { HydratedDocument, Types } from "mongoose";
import { Room } from "src/rooms/room.model";

@Schema({ strict: true, timestamps: true })
export class Reservation {
  @Prop({ type: Types.ObjectId, ref: Room.name, required: true })
  public room!: Types.ObjectId;

  @Prop({
    type: Date,
    required: true,
    min: new Date(date.setHours(0, 0, 0, 0)),
    set: (date: Date) => new Date(date.setHours(0, 0, 0, 0)),
    validate: {
      validator: (date: Date) => date >= new Date(),
      message: "Date can't be in the past",
    },
  })
  public checkInDate!: Date;

  @Prop({
    type: Date,
    required: true,
    min: new Date(date.setHours(0, 0, 0, 0)),
    set: (date: Date) => new Date(date.setHours(0, 0, 0, 0)),
    validate: {
      validator: (date: Date) => date >= new Date(),
      message: "Date can't be in the past",
    },
  })
  public checkOutDate!: Date;
}

export type ReservationDocument = HydratedDocument<Reservation>;
export const ReservationSchema = SchemaFactory.createForClass(Reservation);

// Index for efficient queries on room and dates (non-unique for ranges)
ReservationSchema.index({
  room: 1,
  checkInDate: 1,
  checkOutDate: 1,
});
// Pre-save hook example for validation
ReservationSchema.pre("save", function (next) {
  if (this.checkInDate >= this.checkOutDate) {
    return next(new Error("Start date must be before end date"));
  }
  next();
});
```

</details>

---

Create DTO

```ts
import { IsDate, IsMongoId, IsOptional, IsString, MinDate } from "class-validator";
import { Type } from "class-transformer";

export class CreateReservationDto {
	@IsMongoId()
	room: string;

	@IsDate()
	@Type(() => Date)
	@MinDate(new Date(date.setHours(0, 0, 0, 0), { message: "Check-in date cannot be in the past" })
	checkInDate: Date;

	@IsDate()
	@Type(() => Date)
	@MinDate(new Date(date.setHours(0, 0, 0, 0), { message: "Check-out date cannot be in the past" })
	checkOutDate: Date;
}
```

---

Update DTO

```ts
import { PartialType } from "@nestjs/mapped-types";
import { CreateReservationDto } from "./create-reservation.dto";

export class UpdateReservationDto extends PartialType(CreateReservationDto) {}
```

---

Response DTO

```ts
import { Expose, Transform } from "class-transformer";

export class ReservationResponseDto {
  @Expose()
  id: string;

  @Expose()
  @Transform(({ obj }) => obj.room?.toString())
  room: string;

  @Expose()
  @Transform(({ value }) => value.toISOString().split("T")[0])
  checkInDate: string;

  @Expose()
  @Transform(({ value }) => value.toISOString().split("T")[0])
  checkOutDate: string;

  @Expose()
  createdAt: Date;

  @Expose()
  updatedAt: Date;

  @Expose()
  @Transform(({ obj }) => {
    const checkIn = new Date(obj.checkInDate);
    const checkOut = new Date(obj.checkOutDate);
    const timeDiff = checkOut.getTime() - checkIn.getTime();
    return Math.ceil(timeDiff / (1000 * 3600 * 24));
  })
  nightsCount: number;
}
```

---

Service

```ts
import {
  Injectable,
  ConflictException,
  NotFoundException,
  BadRequestException,
} from "@nestjs/common";
import { InjectModel } from "@nestjs/mongoose";
import { Model, Types } from "mongoose";
import { Reservation } from "./reservation.model";
import { CreateReservationDto } from "./dto/create-reservation.dto";
import { UpdateReservationDto } from "./dto/update-reservation.dto";

@Injectable()
export class ReservationsService {
  public constructor(
    @InjectModel(Reservation.name)
    private readonly reservationModel: Model<ReservationDocument>
  ) {}

  private normalizeDate(date: Date): Date {
    return new Date(date.setHours(0, 0, 0, 0));
  }

  async checkRoomAvailability(
    roomId: string,
    checkInDate: Date,
    checkOutDate: Date,
    excludeReservationId?: string
  ): Promise<boolean> {
    const normalizedCheckIn = this.normalizeDate(new Date(checkInDate));
    const normalizedCheckOut = this.normalizeDate(new Date(checkOutDate));

    const query: any = {
      room: new Types.ObjectId(roomId),
      $or: [
        {
          checkInDate: { $lt: normalizedCheckOut },
          checkOutDate: { $gt: normalizedCheckIn },
        },
      ],
    };

    if (excludeReservationId) {
      query._id = { $ne: new Types.ObjectId(excludeReservationId) };
    }

    const conflictingReservation = await this.reservationModel.findOne(query);
    return !conflictingReservation;
  }

  async create(
    createReservationDto: CreateReservationDto
  ): Promise<ResponseReservationDto> {
    const { room, checkInDate, checkOutDate } = createReservationDto;

    // Валидация дат
    if (checkInDate >= checkOutDate) {
      throw new BadRequestException(
        "Check-out date must be after check-in date"
      );
    }

    // Проверка доступности
    const isAvailable = await this.checkRoomAvailability(
      room,
      checkInDate,
      checkOutDate
    );
    if (!isAvailable) {
      throw new ConflictException(
        "Room is not available for the selected dates"
      );
    }

    const reservation = new this.reservationModel(createReservationDto);
    // !!! + transformation
    return await reservation.save();
  }

  async findAll(): Promise<ResponseReservationDto[]> {
    return this.reservationModel.find().populate("room").exec();
  }

  async findByRoom(roomId: string): Promise<ResponseReservationDto[]> {
    return this.reservationModel
      .find({ room: new Types.ObjectId(roomId) })
      .sort({ checkInDate: 1 })
      .exec();
  }

  async findOne(id: string): Promise<Reservation> {
    // new Types.ObjectId(id), /* ??? */
    const reservation = await this.reservationModel
      .findById(id)
      .populate("room")
      .exec();

    if (!reservation) {
      throw new NotFoundException(`Reservation with ID ${id} not found`);
    }
    // !!! + transformation
    return reservation;
  }

  async update(
    id: string,
    updateReservationDto: UpdateReservationDto
  ): Promise<ResponseReservationDto> {
    const existingReservation = await this.findOne(id);

    const { room, checkInDate, checkOutDate } = updateReservationDto;

    // Если обновляются даты или комната - проверяем доступность
    if (room || checkInDate || checkOutDate) {
      const finalRoom = room || existingReservation.room.toString();
      const finalCheckIn = checkInDate || existingReservation.checkInDate;
      const finalCheckOut = checkOutDate || existingReservation.checkOutDate;

      if (finalCheckIn >= finalCheckOut) {
        throw new BadRequestException(
          "Check-out date must be after check-in date"
        );
      }

      const isAvailable = await this.checkRoomAvailability(
        finalRoom,
        finalCheckIn,
        finalCheckOut,
        id
      );

      if (!isAvailable) {
        throw new ConflictException(
          "Room is not available for the selected dates"
        );
      }
    }

    const updatedReservation = await this.reservationModel
      .findByIdAndUpdate(
        // new Types.ObjectId(id), /* ??? */
        id,
        {
          ...(room && { room: new Types.ObjectId(room) }),
          ...(checkInDate && { checkInDate }),
          ...(checkOutDate && { checkOutDate }),
        },
        { new: true, runValidators: true }
      )
      .populate("room")
      .exec();

    if (!updatedReservation) {
      throw new NotFoundException(`Reservation with ID ${id} not found`);
    }
    // !!! + transformation
    return updatedReservation;
  }

  async remove(id: string): Promise<ResponseReservationDto | null> {
    const result = await this.reservationModel.findByIdAndDelete(id).exec();
    if (!result) {
      throw new NotFoundException(`Reservation with ID ${id} not found`);
    }
    // !!! + transformation
    return result;
  }

  async getRoomAvailability(
    roomId: string,
    startDate?: Date,
    endDate?: Date
  ): Promise<{ available: boolean; conflictingDates?: Date[] }> {
    const normalizedStart = startDate
      ? this.normalizeDate(new Date(startDate))
      : this.normalizeDate(new Date());
    const normalizedEnd = endDate
      ? this.normalizeDate(new Date(endDate))
      : new Date(normalizedStart.getTime() + 30 * 24 * 60 * 60 * 1000);

    const conflicts = await this.reservationModel
      .find({
        room: new Types.ObjectId(roomId),
        $or: [
          {
            checkInDate: { $lt: normalizedEnd },
            checkOutDate: { $gt: normalizedStart },
          },
        ],
      })
      .sort({ checkInDate: 1 })
      .exec();

    return {
      available: conflicts.length === 0,
      conflictingDates: conflicts.map((res) => res.checkInDate),
    };
  }
}
```

---

Controller

```ts
import {
  Controller,
  Get,
  Post,
  Body,
  Patch,
  Param,
  Delete,
  UsePipes,
  ValidationPipe,
} from "@nestjs/common";
import { ReservationsService } from "./reservations.service";
import { CreateReservationDto } from "./dto/create-reservation.dto";
import { UpdateReservationDto } from "./dto/update-reservation.dto";
import { ReservationResponseDto } from "./dto/reservation-response.dto";
import { Serialize } from "../interceptors/serialize.interceptor";

@Controller("reservations")
@Serialize(ReservationResponseDto)
export class ReservationsController {
  constructor(private readonly reservationsService: ReservationsService) {}

  @Post()
  @UsePipes(new ValidationPipe({ transform: true }))
  async create(
    @Body() createReservationDto: CreateReservationDto
  ): Promise<ReservationDocument | null> {
    return await this.reservationsService.create(createReservationDto);
  }

  @Get()
  async findAll(): Promise<ReservationDocument[]> {
    return await this.reservationsService.findAll();
  }

  @Get("room/:roomId")
  async findByRoom(
    @Param("roomId") roomId: string
  ): Promise<ResponseReservationDto[]> {
    return await this.reservationsService.findByRoom(roomId);
  }

  @Get("availability/:roomId")
  async checkAvailability(
    @Param("roomId") roomId: string,
    @Body("startDate") startDate?: Date,
    @Body("endDate") endDate?: Date
  ): Promise<{ available: boolean; conflictingDates?: Date[] }> {
    return await this.reservationsService.getRoomAvailability(
      roomId,
      startDate,
      endDate
    );
  }

  @Get(":id")
  async findOne(@Param("id") id: string): Promise<ReservationDocument | null> {
    return await this.reservationsService.findOne(id);
  }

  @Patch(":id")
  @UsePipes(new ValidationPipe({ transform: true }))
  async update(
    @Param("id") id: string,
    @Body() updateReservationDto: UpdateReservationDto
  ): Promise<ReservationDocument | null> {
    return await this.reservationsService.update(id, updateReservationDto);
  }

  @Delete(":id")
  async remove(@Param("id") id: string): Promise<ReservationDocument | null> {
    return await this.reservationsService.remove(id);
  }
}
```

---

Serialize Interceptor for Responses

```ts
import {
  UseInterceptors,
  NestInterceptor,
  ExecutionContext,
  CallHandler,
} from "@nestjs/common";
import { Observable } from "rxjs";
import { map } from "rxjs/operators";
import { plainToInstance } from "class-transformer";

export function Serialize(dto: any) {
  return UseInterceptors(new SerializeInterceptor(dto));
}

export class SerializeInterceptor implements NestInterceptor {
  constructor(private dto: any) {}

  intercept(context: ExecutionContext, handler: CallHandler): Observable<any> {
    return handler.handle().pipe(
      map((data: any) => {
        return plainToInstance(this.dto, data, {
          excludeExtraneousValues: true,
        });
      })
    );
  }
}
```

<!-- <details>
<summary>Response serializer</summary>

```ts
import {
  CallHandler,
  ExecutionContext,
  NestInterceptor,
  UseInterceptors,
} from "@nestjs/common";
import { Observable } from "rxjs";
import { map } from "rxjs/operators";
import { plainToInstance } from "class-transformer";

interface ClassConstructor {
  new (...args: any[]): {};
}

export function Serialize(dto: ClassConstructor) {
  return UseInterceptors(new SerializeInterceptor(dto));
}

export class SerializeInterceptor implements NestInterceptor {
  constructor(private dto: any) {}

  intercept(context: ExecutionContext, handler: CallHandler): Observable<any> {
    return handler.handle().pipe(
      map((data: any) => {
        if (data && data.rooms) {
          // Для пагинированных результатов
          return {
            ...data,
            rooms: plainToInstance(this.dto, data.rooms, {
              excludeExtraneousValues: true,
            }),
          };
        }

        return plainToInstance(this.dto, data, {
          excludeExtraneousValues: true,
        });
      })
    );
  }
}
```

</details> -->
