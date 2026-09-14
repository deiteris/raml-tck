# Contributing

Each test case must contain the following files:
* A valid `*valid*.raml` file showcasing valid use of the feature under test
* An invalid `*invalid*.raml` files showcasing invalid use of the feature under test

and optionally:
* Any number of Library and Fragment `.raml` files as well as `.json`, `.xml`, `.md`, and any other files in either the root folder or its subtree. These files must be used by the main valid/invalid `.raml` files.

Test cases must be put under the `tests` directory by either putting your tests into an existing folder or creating a new one. Hereby, it is important that the folder name does reflect the main purpose and each file name should also be meaningful. For example, if you want to contribute a new test case that covers traits in RAML 1.0, you only need to create a new folder for your test case under `tests/raml-1.0/Traits` and copy your RAML files into that. 

## Running tests

Upstream ran the suite through [raml-tck-runner](https://github.com/raml-org/raml-tck-runner). That project is archived along with the runners this fork removed, so run a new test case against whichever processor you are working on, and record its expected outcome there.
