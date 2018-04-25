Contibuting
-----------
`back-lite` was written with the goal of code being easy to read
at a glance, (assuming the reader is familiar with bash) while 
still being easy to modify and extend as needed.  Contributions 
should follow the style of the existing codebase. 

In particular:

  - 80 columns, no exceptions
  - 2 spaces for indentation
  - camelCase variable/function names
  - Prefer short-circuiting with `&&` and `||` over 
    `if ...; then ... ; fi` blocks for small conditional commands
  - Prefer `[[ ... ]]` over `[ ... ]` or `test ...`

Some desireable areas for contributions are:

  - New target types
  - Improved configurability
  - Better validation of user input
  - Automated tests

Contributions should be linted using [shellcheck], and new features 
should be fully tested.

[shellcheck]: https://github.com/koalaman/shellcheck
