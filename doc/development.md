# Octave Packages development guide

## Quick info

Your package management tool can read the **entire package index at once**
as Octave array of struct into the variable `package_index` using the command:
```
function package_index = package_index_resolve ()
  package_index = jsondecode (urlread ("https://packages.octave.org/packages.json"), "makeValidName", false);
endfunction
```
Depending on your internet connection, this is an inexpensive operation,
thus a caching strategy is probably not necessary.
```
>> tic; package_index = package_index_resolve (); toc
Elapsed time is 0.820996 seconds.
```


## Detailed information for writing a package management tool

This package index is build from the perspective of a package management tool
written in the GNU Octave language.


### Read the package index

Using the routine `package_index_resolve()` as describe in the quick info above,
an Octave array of struct `package_index` indexed by the package names is returned.

To get the first three packages names, for example, type:
```
>> fieldnames (package_index)(1:3)
ans =
{
  [1,1] = arduino
  [2,1] = audio
  [3,1] = bim
}
```


### Extract package details

After reading the package index to the variable `package_index` as shown above,
one can obtain more detailed information about individual packages.
The following code shows all available struct fields for `pkg-example`:
```
>> fieldnames (package_index.("pkg-example"))
ans =
{
  [1,1] = name
  [2,1] = description
  [3,1] = icon
  [4,1] = links
  [5,1] = maintainers
  [6,1] = versions
}
```
Similar, one can see all details of the latest `pkg-example` version:
```
>> package_index.("pkg-example").versions(1)
ans =

  scalar structure containing the fields:

    id = 1.1.0
    date = 2021-04-06
    sha256 = bff441755f0d68596f2efd027fe637b5b6c52b722ffd6255bdb8a5f34ab4ef2a
    url = https://github.com/gnu-octave/pkg-example/archive/1.1.0.tar.gz
    depends =
    {
      [1,1] = octave (>= 4.0.0)
      [2,1] = pkg
    }
```


### Compatibility with Octave's `pkg` tool

In case you want to stay compatible with
[Octaves builtin package management tool `pkg`](https://www.octave.org/doc/v6.2.0/XREFpkg.html)
you should care about the following default settings:

- `pkg prefix`: directory new packages are installed to.
  ```
  >> pkg prefix -global
  Installation prefix:             /usr/share/octave/packages
  Architecture dependent prefix:   /usr/lib/octave/packages

  >> pkg prefix -local
  Installation prefix:             /home/username/octave
  Architecture dependent prefix:   /home/username/octave
  ```

- `pkg local_list`  (default: `/home/username/.octave_packages`):
- `pkg global_list` (default: `/usr/share/octave/octave_packages`):
  File listing installed local or global packages.

  The file is simply created by saving an array of struct
  using Octave's default `save` command:
  ```
  >> load /home/username/.octave_packages
  >> local_packages
  local_packages =
  {
    [1,1] =

      scalar structure containing the fields:

        name = pkg_example
        version = 1.0.0
        date = 2020-09-02
        author = Kai T. Ohlhus <k.ohlhus@gmail.com>
        maintainer = Kai T. Ohlhus <k.ohlhus@gmail.com>
        title = Minimal example package to demonstrate the Octave package extensions.
        description = Minimal example package to demonstrate the Octave package  extensions.  It shows how to organize Octave, C/C++, and FORTRAN code within  a package and to properly compile it.
        depends =
        dir = /home/siko1056/octave/pkg_example-1.0.0
        archprefix = /home/siko1056/octave/pkg_example-1.0.0

  }
  ```


## Developing the websites of GNU Octave packages

Two important links first:

1. Development repository: <https://github.com/gnu-octave/packages/>
2. Deployment URL: <https://gnu-octave.github.io/packages/>

In this section some hints for developing this index from the server site are
given.

- The whole index is a **static website** generated by
  [bundler](https://bundler.io/) and [Jekyll](https://jekyllrb.com/)
  and hosted on
  [GitHub pages](https://pages.github.com/).

- There is **no limitation** to host the generated static pages on any other
  **arbitrary webserver**.

- Build the static websites with bundler/Jekyll
  ```
  git clone https://github.com/gnu-octave/packages
  cd packages
  bundle install
  bundle exec jekyll build
  ```
  and copy the content of the generated `_site` directory to the webserver of
  your choice.

- Basically, there are two layouts.
  One for the package index `_layouts/package_index.html` (used by
  `assets/index.md`) and one for the individual packages pages
  `_layouts/package.html` (used by all `.md` files in the `package` directory).

- The index read by `package_index_resolve()` is generated from
  `packages/index.md`.

- The package index page uses following JavaScript/CSS frameworks:
  - [FontAwesome](https://fontawesome.com/) for awesome fonts.
  - [bootstrap](https://getbootstrap.com/): for simply good-looking design.
  - [DataTables](https://datatables.net/) for a dynamic search feature.
    Configuration at <https://datatables.net/download/index>:
    ```
    Styling framework
      [x] DataTables
    Packages
      [x] DataTables
    Extensions
      [x] Scroller
    Download method: CDN
      [x] Minify
      [x] Concatenate
    ```
    If no JavaScript is available, a static HTML table is displayed.
  - [jQuery](https://jquery.com/): needed by bootstrap and DataTables.

- The individual package pages are generated by filling the layout with the
  metadata given in YAML format,
  as described in [CONTRIBUTING.md](/CONTRIBUTING.md).
