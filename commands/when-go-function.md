---
description: Create Go CLI function with architecture requirements
argument-hint: [task description]
---

$ARGUMENTS

When implementing this, follow these architecture requirements:

## Project Structure
- Place functions in a `functions/` folder that is peered with `go.mod`
- This could be at the top level or in a subdirectory if in a monorepo

## CLI Framework (urfave/cli/v2)
- Use `urfave/cli/v2` for CLI commands
- Define the action as a **top-level function** rather than inlined
- Action function naming:
  - If the command has no name (root command), call the function `action`
  - If the command has a name, use `action{CommandName}` (e.g., `actionDeploy`, `actionBuild`)
- All CLI calls must go to the action function - no shortcuts for lambdas
- Example:
  ```go
  func action(c *cli.Context) error {
      // root command implementation
  }

  func actionDeploy(c *cli.Context) error {
      // deploy subcommand implementation
  }

  &cli.Command{
      Name: "deploy",
      Action: actionDeploy,  // top-level func, not inline
  }
  ```

## Environment Variables
- **Always** read environment variables through the CLI context
- **Never** call `os.Getenv` directly
- Always include an `"env"` string flag with `ENV` environment variable mapping
- Example:
  ```go
  &cli.Command{
      Flags: []cli.Flag{
          &cli.StringFlag{
              Name:    "env",
              EnvVars: []string{"ENV"},
              Value:   "development",
          },
          // other flags...
      },
  }
  ```

## Dependency Injection
- Check if a `di/` package exists peered with `go.mod`
- If present:
  - Initialize the DI container
  - Attempt to wire objects through DI rather than initializing them directly
  - Use the DI container for dependency resolution
