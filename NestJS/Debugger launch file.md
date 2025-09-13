```jsonc
{
	"version": "0.2.0",
	"configurations": [
		{
			"type": "node",
			"request": "launch",
			"name": "Launch NestJS",
			"runtimeArgs": ["-r", "ts-node/register", "-r", "tsconfig-paths/register"],
			"args": ["src/main.ts"],
			"autoAttachChildProcesses": true,
			"outFiles": ["${workspaceFolder}/dist/**/*.js"],
			"sourceMaps": true,
			"env": {
				"PORT": "3000" // Set your desired port here
			},
			"console": "integratedTerminal"
		}
	]
}
```
