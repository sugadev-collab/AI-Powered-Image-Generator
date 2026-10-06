# 03 — Coding Guidelines

These apply everywhere in the project.

## Comments and explanations

- Write a short, nice explanation as a comment **above each function**.
- **Explain any mathematical formula** above its code.
- Keep code readable: do **not** comment every line. Put longer explanations at the **top of the file**, clearly marked as "Read this before reading the code".

## Structure and performance

- Keep code clean and structured, and reusable where possible.
- Where reuse hurts performance, duplicating code is acceptable. **Performance is preferred over looks.**

## Libraries

- Libraries are acceptable, but they must not be big or unnecessary for the project.

## C and C++

- Write **pure C++**.
- C may be used inside it when it makes a part much faster. It must then be marked as C code in a comment.
- Avoid interfaces between C and C++. Use C only where it gives extremely high performance over C++, especially in libraries.