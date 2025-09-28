Perform commont settings

---

# 4. Install and Configure Dependencies

### dotenv [(docs)](https://www.npmjs.com/package/dotenv)

<details>
<summary>Settings</summary>
<dl><dd>

Install:

```sh
npm install dotenv --save
```

Create and configure a `.env` file:

```sh
echo "" > .env
```

</dd></dl>
</details>

### Docker + MongoDB

<details>
<summary>Settings</summary>
<dl><dd>

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

</dd></dl>
</details>

### ...

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
			envFilePath: "./.env",
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
<summary><strong>Generate rooms feature</strong></summary>
<dl><dd>

You can generate the feature template with a single command and make a few adjustments:

```sh
nest generate resource rooms
```

```sh
rm -rf ./src/rooms/entities
```

```sh
echo "" > ./src/rooms/dto/response-room.dto.ts
```

```sh
echo "" > ./src/rooms/room.model.ts
```

```sh
echo "" > ./src/rooms/dto/filter-room.dto.ts
```

Or you can create the files manually using separate commands.

```sh
nest generate module rooms
```

```sh
nest generate service rooms
```

```sh
nest generate controller rooms
```

```sh
nest generate class rooms/dto/create-room.dto --flat
```

```sh
nest generate class rooms/dto/update-room.dto --flat
```

```sh
nest generate class rooms/dto/response-room.dto --flat
```

```sh
echo "" > ./src/rooms/room.model.ts
```

```sh
nest generate class rooms/dto/filter-room.dto --flat
```

room: Don’t add any content yet; just ensure the files are created.

- `src/rooms/rooms.service.ts` file
- `src/rooms/rooms.controller.ts` file
- `src/rooms/rooms.module.ts` file
- `src/rooms/dto/create-room.dto.ts` file
- `src/rooms/dto/update-room.dto.ts` file
- `src/rooms/dto/response-room.dto.ts` file
- `src/rooms/room.model.ts` file
- `src/rooms/dto/filter-room.dto.ts` file

</dd></dl>
</details>

<details>
<summary><strong>Generate reservations feature</strong></summary>
<dl><dd>

You can generate the feature template with a single command and make a few adjustments:

```sh
nest generate resource reservations
```

```sh
rm -rf ./src/reservations/entities
```

```sh
echo "" > ./src/reservations/dto/response-reservation.dto.ts
```

```sh
echo "" > ./src/reservations/reservation.model.ts
```

```sh
echo "" > ./src/reservations/dto/filter-reservation.dto.ts
```

Or you can create the files manually using separate commands.

```sh
nest generate module reservations
```

```sh
nest generate service reservations
```

```sh
nest generate controller reservations
```

```sh
nest generate class reservations/dto/create-reservation.dto --flat
```

```sh
nest generate class reservations/dto/update-reservation.dto --flat
```

```sh
nest generate class reservations/dto/response-reservation.dto --flat
```

```sh
echo "" > ./src/reservations/reservation.model.ts
```

```sh
nest generate class reservations/dto/filter-reservation.dto --flat
```

reservation: Don’t add any content yet; just ensure the files are created.

- `src/reservations/reservations.service.ts` file
- `src/reservations/reservations.controller.ts` file
- `src/reservations/reservations.module.ts` file
- `src/reservations/dto/create-reservation.dto.ts` file
- `src/reservations/dto/update-reservation.dto.ts` file
- `src/reservations/dto/response-reservation.dto.ts` file
- `src/reservations/reservation.model.ts` file
- `src/reservations/dto/filter-reservation.dto.ts` file

</dd></dl>
</details>

<br/>

### Create the model, schema, document

<details>
<summary>src/rooms/room.model.ts</summary>

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

// Nested schema
@Schema({ _id: false })
export class Amenity {
	@Prop({ required: true, trim: true })
	public name!: string;

	@Prop({ default: false })
	public isPremium?: boolean;

	@Prop()
	public description?: string;
}

// Main schema
@Schema({ _id: false })
export class Pricing {
	@Prop({ required: true, min: 0 })
	public basePrice!: number;

	@Prop({ min: 0 })
	public weekendSurcharge?: number;

	@Prop({ min: 0 })
	public seasonSurcharge?: number;
}

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

	@Prop({
		type: Number,
		min: 1,
		max: 6,
	})
	public maxOccupancy!: number;

	@Prop({
		type: [Amenity],
		validate: {
			validator: (arr: Amenity[]): boolean => arr.length <= 5,
			message: "Amenity limit is 5",
		},
	})
	public amenities!: Amenity[];

	@Prop({ type: Pricing, required: true })
	public pricing!: Pricing;

	@Prop({ type: Date })
	public lastMaintenance?: Date;

	@Prop({ type: Date, default: Date.now })
	public nextMaintenance!: Date;

	/* Virtual fields */
	public get displayName(): string {
		return `Room ${this.roomNumber} (${this.roomType})`;
	}
	public get isAvailable(): boolean {
		return this.status === RoomStatus.AVAILABLE;
	}

	/* Methods */
	public hasAmenity(amenityName: string): boolean {
		return this.amenities?.some((a): boolean => a.name === amenityName) ?? false;
	}

	/* Static */
	public static async findByType(this: RoomModelType, roomType: RoomType): Promise<RoomDocument[]> {
		return this.find({ roomType }).exec();
	}
}
export type RoomModelType = Model<RoomDocument> & typeof Room;
export type RoomDocument = HydratedDocument<Room>;
export const RoomSchema = SchemaFactory.createForClass(Room);

```

</details>

### Add DTOs

#### CreateDto

<details>
<summary>src/rooms/dto/create-room.dto.ts</summary>

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

export class AmenityDto {
  @IsBoolean()
  @IsOptional()
  public isPremium?: boolean;

  @IsOptional()
  public description?: string;
}

export class PricingDto {
  @IsNumber()
  @Min(0)
  public basePrice!: number;

  @IsNumber()
  @Min(0)
  @IsOptional()
  public weekendSurcharge?: number;

  @IsNumber()
  @Min(0)
  @IsOptional()
  public seasonSurcharge?: number;
}

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

  @IsInt()
  @Min(1)
  @Max(6)
  public maxOccupancy!: number;

  @IsArray()
  @ValidateNested({ each: true })
  @Type((): typeof AmenityDto => AmenityDto)
  @IsOptional()
  public amenities?: AmenityDto[];

  @ValidateNested()
  @Type((): typeof PricingDto => PricingDto)
  public pricing!: PricingDto;

  @IsDate()
  @Type((): typeof Date => Date)
  @IsOptional()
  public lastMaintenance?: Date;

  @IsDate()
  @Type((): typeof Date => Date)
  @IsOptional()
  public nextMaintenance?: Date;
}
```

</details>

#### UpdateDto

<details>
<summary>src/rooms/dto/update-room.dto.ts</summary>

```ts
import { PartialType } from "@nestjs/mapped-types";
import { CreateRoomDto } from "./create-room.dto";

export class UpdateRoomDto extends PartialType(CreateRoomDto) {}
```

</details>

#### ResponseDto

<details>
<summary>src/rooms/dto/response-room.dto.ts</summary>

```ts
import { Exclude, Expose, Transform } from "class-transformer";
import { RoomStatus, RoomType } from "../room.model";
// import { RoomType, RoomStatus } from "../schemas/room.schema";

export class AmenityResponseDto {
  @Expose()
  public name!: string;

  @Expose()
  public isPremium?: boolean;

  @Expose()
  public description?: string;
}

export class PricingResponseDto {
  @Expose()
  public basePrice!: number;

  @Expose()
  public weekendSurcharge?: number;

  @Expose()
  public seasonSurcharge?: number;
}

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

  @Expose()
  public maxOccupancy!: number;

  @Expose()
  @Transform((params): AmenityResponseDto[] => params.value || [])
  public amenities!: AmenityResponseDto[];

  @Expose()
  public pricing!: PricingResponseDto;

  @Expose()
  public lastMaintenance?: Date;

  @Expose()
  public nextMaintenance!: Date;

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

#### FilterDto

<details>
<summary>src/rooms/dto/filter-room.dto.ts</summary>

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

  @IsInt()
  @Min(1)
  @Max(6)
  @IsOptional()
  @Type((): typeof Number => Number)
  public minOccupancy?: number;

  @IsInt()
  @Min(0)
  @IsOptional()
  @Type((): typeof Number => Number)
  public maxPrice?: number;

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

### Register the MongooseModule `forFeature` in the `imports` Array of the Feature Module

<details>
<summary>src/notes/notes.module.ts</summary>

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

### Inject the Feature Model into the Feature Service

<details>
<summary>src/notes/notes.service.ts</summary>

```TypeScript
import { ConflictException, Injectable, NotFoundException } from "@nestjs/common";
import { InjectModel } from "@nestjs/mongoose";
import { RootFilterQuery, Types } from "mongoose";
import { Room, RoomDocument, RoomStatus, RoomType, type RoomModelType } from "./room.model";
import { CreateRoomDto } from "./dto/create-room.dto";
import { UpdateRoomDto } from "./dto/update-room.dto";
import { RoomFilterDto } from "./dto/filter-room.dto";

@Injectable()
export class RoomsService {
	public constructor(@InjectModel(Room.name) private readonly roomModel: RoomModelType) {}

	public async create(dto: CreateRoomDto): Promise<RoomDocument> {
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

		return roomInstance;
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
		if (dto.roomNumber) {
			const existingRoom = await this.roomModel.findOne({
				roomNumber: dto.roomNumber,
				_id: { $ne: new Types.ObjectId(id) },
			});

			if (existingRoom) {
				throw new ConflictException(`Room with number ${dto.roomNumber} already exists`);
			}
		}

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

<!-- ### Add interceptor -->

<!--
```sh
mkdir src/rooms/interceptors
```

```sh
echo "" > src/rooms/interceptors/serialize.interceptor.ts
```

<details>
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

### Controller

<details>
<summary>controller</summary>

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

@Controller("rooms")
@UseInterceptors(new ClassSerializerInterceptor(RoomResponseDto))
export class RoomController {
  public constructor(private readonly roomService: RoomsService) {}

  @Post()
  public async create(
    @Body() createRoomDto: CreateRoomDto
  ): Promise<RoomDocument> {
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

### Add Decorators to DTOs

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

### Do the rest (CRUD, service, controller, etc.)

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
