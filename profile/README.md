# rotini

Rotini is a spec-driven CLI package for Go, with a codegen tool and a runtime library. You declare your commands, flags and arguments in a spec file (YAML, JSON, JSONC or TOML). Rotini validates it and generates the typed Go, the command tree and a handler stub per command. You write each command's handler, and the runtime parses and validates input before calling it. The loop is: edit the spec, go generate, provide handler implementations, build.

For more information, see the [rotini](https://rotini.dev) documentation.
