claude -p "Review the staged/uncommitted changes in this Rust project for defensive programming issues. Check for:

**Code Smells to Flag:**
1. **Vector Indexing** - Direct indexing like `vec[0]` or `slice[i]` that could panic. Suggest slice pattern matching: `match items.as_slice() { [first] => ..., [] => ... }`

2. **Lazy Default** - Using `..Default::default()` which hides fields and misses compiler checks when new fields are added. Suggest explicit field initialization or destructuring the default.

3. **Fragile Trait Impls** - `PartialEq`, `Hash`, `Debug`, `Clone` implementations that don't destructure the struct, missing new fields when they're added. Suggest: `let Self { field1, field2, .. } = self;`

4. **From That Should Be TryFrom** - `From` implementations using `unwrap()`, `unwrap_or()`, `unwrap_or_else()`, or `expect()`. These should be `TryFrom` to make fallibility explicit.

5. **Non-Exhaustive Matches** - Using `_ => {}` catch-all patterns in match statements. Suggest explicit variant listing so compiler warns on new variants.

6. **Anonymous Placeholders** - Using bare `_` in destructuring patterns like `Self::Foo { _, _, .. }`. Suggest named ignores: `has_fuel: _, has_crew: _`

7. **Missing Temporary Mutability** - Mutable variables that could be shadowed as immutable after initialization: `let mut x = ...; x.sort(); let x = x;`

8. **Unprotected Constructors** (for library code) - Public structs with all public fields that should enforce validation. Suggest `_private: ()` field or `#[non_exhaustive]`.

9. **Missing #[must_use]** - Types or functions whose return values should not be ignored (configs, guards, results).

10. **Boolean Parameters** - Functions with multiple bool params like `fn foo(x: bool, y: bool)`. Suggest descriptive enums instead.

**Relevant Clippy Lints to Suggest:**
- clippy::indexing_slicing
- clippy::fallible_impl_from  
- clippy::wildcard_enum_match_arm
- clippy::fn_params_excessive_bools
- clippy::must_use_candidate

Only report actual issues found. Be specific about line numbers and provide concrete fix suggestions." --allowedTools bash,Read,Grep
