```sh
npm install --save @nestjs/config
```

```sh
npm install --save joi
```

---

#### `src/config/configuration.ts`

```TypeScript
import * as Joi from "joi";
export type EnvironmentVariables = {
	HOST: string;
	PORT: number;
};

export const validationSchema = Joi.object<EnvironmentVariables, true>({
	PORT: Joi.number().required(),
	HOST: Joi.string().required(),
});

```

---

#### `src/app.module.ts`

```TypeScript
import { Module } from "@nestjs/common";
import { AppController } from "./app.controller";
import { AppService } from "./app.service";
// ...
import { ConfigModule } from "@nestjs/config";
import { validationSchema } from "./config/configuration";

@Module({
	imports: [
		ConfigModule.forRoot({
			envFilePath: ".env",
			skipProcessEnv: true,
			isGlobal: false,

			validationSchema: validationSchema,

			validationOptions: {
				allowUnknown: true,
				abortEarly: true,
			},
		}),
    // ...
	],
	controllers: [AppController],
	providers: [AppService],
})
export class AppModule {}

```

---

#### `src/page/page.controller.ts`

```TypeScript
// ...
import { ConfigService } from "@nestjs/config";
import { EnvironmentVariables } from "src/config/configuration";

@Controller("page")
export class PageController {
	public constructor(private readonly config: ConfigService<EnvironmentVariables, true>) {}

	@Get("list")
	public list(): Array<string> {
		const value = this.config.get("PORT", { infer: true });
		const value2 = this.config.get("HOST", { infer: true });
    // ...
	}
```
