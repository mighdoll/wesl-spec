# Packaging

(TBD)

The section will discuss packaging extended wgsl in reusable crates or npm packages.

See also [Visiblity Control](Visiblity.md).


* What goes in `package.json`?
  * at runtime in the browser, the linker needs
    wgsl strings and relative paths for every needed wgsl file, from every every package.
    * (this so the linker can resolve imports)
  * something needs to tell javascript bundlers to include needed wgsl files in the bundle:
    * we could ask package publishers to produce a javascript file containing a map of relative paths and wgsl strings
    * perhaps better would be to have a vite/rollup plugin trace from the source wgsl to the needed package wgsl files (using the linker as a library to parse wgsl)
      * this would avoid requiring publishers to manually publish string maps of wgsl text, 
        instead publishers would just publish files in dist. 
        * the consumer plugin tool would bundle files into strings.
      * corner case issue: runtime conditional compilation could potentially change the imported
        files required. Perhaps we'll need some workaround markers in this rare case, so tree shaking can still work.
    * note that the wgsl-linker already has this problem of bundling local wgsl files, 
      and asks users to 'self package' wgsl using import.meta.glob
      (which bundles files into [file path, wgsl string] pairs)
  * what other metadata should be available to the linker?
    * name of package.
    * host visible bits from wgsl? e.g. entry points, runtime variables, overrides, binding groups, uniforms?
      An IDE tool could use that data to typecheck/autocomplete host calls.
      Things like entry points are computable from the wgsl,
      but it's a lot to ask of a language server to parse through all the wgsl in advance to find them.
      Probably better to put it into the metadata if we need it.
      * these are parts of the API of the package as a whole and can reduce bugs by making them visible
  * how do we handle packages with multiple entry points e.g. stoneberry/reduce stoneberry/prefixSum.
* What goes in `cargo.toml`? 

## `wgsl.toml` file

The `wgsl.toml` file is a configuration file that tells the language server where to find the packages. It is similar to a `Cargo.toml` file in that regard.

```toml
name = "my"
edition = "2024"

[dependencies]
bevy_ui = { cargo = "bevy_ui" }
shader_wiz = { path = "./node_modules/shader_wiz/src/main.wgsl" }
```

We specify how to resolve the paths of packages instead of scanning folders for `*.wgsl` files.
In the Javascript world, it is common to have a `node_modules` folder with 10k files, which is not practical for a language server to scan.

We are planning on taking advantage of existing package managers, such as `cargo` for Rust, and `npm` for Javascript. This makes it easier for users to consume shaders, and makes sense for ecosystem-specific tools.

### Unresolved questions

* Is .toml the best file format for the `wgsl.toml`? Some alternatives would be JSON/JSON5 and StrictYAML.
* do we need to put paths for every package's wgsl in wgsl.toml?
  * Fine to get started that way if need be, but its a maintenance burden every user..
  * Better if we could get the language servers to find the wgsl in packages in node_modules.
* Do we need to list the wgsl package names in wgsl.toml?
  They'll already be listed in as cargo.toml / package.json..
