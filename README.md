# marimo-opencode-skill

Prevent AI agents from breaking marimo notebooks with Python/Jupyter idioms.

## Guardrails, Not a Reference Manual

This skill is designed specifically to keep agents on track when building **interactive notebooks** (the standard use case).

It focuses on preventing DAG errors and broken UI reactivity. It does **not** cover the entire Marimo API, nor does it cover advanced programmatic use cases like `Cell.run()`.

## Why?

LLMs are overfitted on Jupyter notebooks and imperative Python scripts. When they try to write `marimo`, they usually break the reactivity model by:

1.  **Defining functions for cells** (Marimo cells share the global scope; passing args is redundant and breaks the DAG).
2.  **Using `print()`** (Fails silently in `marimo run` / app mode; must use `mo.md` or `mo.stat`).
3.  **Returning values** (Cells don't "return" data; they define global variables).
4.  **Mutating state** (Reading and writing a variable in the same cell creates cycles).

This skill provides context prompts to force the model into "Marimo Mode."

## Enforced Patterns

The skill actively prompts the agent to follow these rules:

| Category | Rule | Reason |
| :--- | :--- | :--- |
| **Variables** | **Global, not Parameter** | Cells are not functions. Don't write `def process(df):`. Just use `df` directly. |
| **Output** | **`mo.md()` / `mo.stat()`** | `print()` outputs go to the console, not the UI, when running as an app. |
| **Flow Control** | **`mo.stop()`, not `raise`** | Use `mo.stop()` to conditionally halt execution without crashing the app UI. |
| **Reactivity** | **Split Definition & Read** | If you define `slider = mo.ui.slider()`, you **cannot** read `slider.value` in the same cell. |
| **Locals** | **Use `_` prefix** | `_var` stays local to the cell. Anything else becomes a global variable in the DAG. |
| **Callbacks** | **`value`, not `change`** | Marimo callbacks receive the raw value (int/str), not a dictionary diff like Jupyter. |
| **SQL** | **Direct DataFrame** | `mo.sql()` returns a DataFrame directly. Do not try to access `.value`. |
| **Forms** | **Check for `None`** | `form.value` is `None` until submitted. Guard against this with `mo.stop()`. |
| **Mutation** | **Mutate Locally** | Do not mutate a global variable (like a DataFrame) in a different cell than where it was defined. |
