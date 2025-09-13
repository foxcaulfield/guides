```jsonc
{
	"version": "0.2.0",
	"configurations": [
		{
			"type": "node",
			"request": "launch",
			"name": "Debug NestJS",
			"runtimeArgs": [
				"-r",
				"ts-node/register",
				"-r",
				"tsconfig-paths/register"
			],
			"args": [
				"-r",
				"tsconfig-paths/register",
				"src/main.ts"
			],
			"console": "integratedTerminal",
			"sourceMaps": true,
			"env": {
				"PORT": "3000" // Set your desired port here
			}
		},
		{
			"type": "node",
			"request": "launch",
			"name": "Alt. Debug NestJS",
			"runtimeExecutable": "npm",
			"runtimeArgs": [
				"run",
				"start:debug",
				"--",
				"--inspect-brk"
			],
			"console": "integratedTerminal",
			"restart": true,
			"autoAttachChildProcesses": true
		}
	]
}


```
