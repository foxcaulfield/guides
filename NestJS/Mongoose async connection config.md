#### `src/app.module.ts`


```TypeScript
@Module({
	imports: [
    ConfigModule.forRoot({
  			envFilePath: ".env",
  			skipProcessEnv: true,
  			isGlobal: false,
  
  			validationSchema: envValidationSchema,
  
  			validationOptions: {
  				allowUnknown: true,
  				abortEarly: true,
  			},
  		}),
  		MongooseModule.forRootAsync({
  			imports: [ConfigModule],
  			useFactory: (configService: ConfigService<EnvironmentVariables, true>): MongooseModuleFactoryOptions => {
  				const mongoRootUsername = configService.get("MONGO_INITDB_ROOT_USERNAME", { infer: true });
  				const mongoRootPassword = configService.get("MONGO_INITDB_ROOT_PASSWORD", { infer: true });
  				const mongoHost = configService.get("MONGO_HOST", { infer: true });
  				const mongoPort = configService.get("MONGO_PORT", { infer: true });
  				const mongoDb = configService.get("MONGO_INITDB_DATABASE", { infer: true });
  
  				return {
  					uri: `mongodb://${mongoRootUsername}:${mongoRootPassword}@${mongoHost}:${mongoPort}/${mongoDb}`,
  				};
  			},
  			inject: [ConfigService],
  		}),
      // ...
```
