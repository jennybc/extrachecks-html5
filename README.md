
<!-- README.md is generated from README.Rmd. Please edit that file -->

## What is this?

R switched to using HTML5 in R 4.2.0, and CRAN has started enforcing
that packages on CRAN generate valid HTML5. In a few cases, roxygen2 was
generating invalid HTML5, so many packages have been notified that they
have issues. **We expect that simply updating roxygen2 to its latest
CRAN version (7.2.1), redocumenting, and then sending your package back
in is enough to fix 99.9% of the issues you might see.**

However, we also know that you might want to validate that the issue is
fixed, so **this repo provides a Github Actions workflow file that you
can use to reproduce the issues and confirm that they have been
resolved. Follow the instructions in the next section to understand how
you can use it.**

The note from CRAN looks something like:

    In particular, please see the "Found the following HTML validation
    problems" NOTEs in the "HTML version of manual" check for the r-devel
    debian checks results.  

Additionally, it would be nice to compile a table of the issues and
whether or not updating roxygen2 fixes the problem. I’ve started the
table below with my specific issue, but feel free to send a PR with your
own specific issues.

## How do I use it?

-   Update roxygen2 to 7.2.1

    ``` r
    install.packages("roxygen2")
    ```

-   Create a branch to fix the HTML5 issues on

    ``` r
    usethis::pr_init("fix/invalid-html5")
    ```

-   Download and commit the Github Actions workflow file that allows you
    to check for this issue.

    ``` r
    usethis::use_github_action(url = "https://github.com/DavisVaughan/extrachecks-html5/blob/main/R-CMD-check-HTML5.yaml")
    ```

    After downloading it, make sure to commit it.

-   Push the workflow file and go ahead and open the PR, the build
    should fail, and should tell you where you have HTML issues.

    ``` r
    usethis::pr_push()
    ```

-   Redocument your package now that you have roxygen2 7.2.1, which
    should fix most issues.

    ``` r
    devtools::document()
    ```

    Commit the changes and push.

-   In 99.9% of cases, just updating roxygen2 is enough. There are some
    cases where you will have to do more work manually. We are
    attempting to capture all issues in the table below. Please open an
    issue or submit a PR if you have an issue that is not yet captured.

-   Once everything is working, remove the workflow file from the PR,
    merge it, and then finish up locally with:

    ``` r
    usethis::pr_finish()
    ```

## Specific issues

| Issue                                                    | Notes                                                                                              | Solved by updating roxygen2? |
|----------------------------------------------------------|----------------------------------------------------------------------------------------------------|------------------------------|
| `Warning: <img> attribute "align" not allowed for HTML5` | The logo in the package doc md file needs to use a `style` attribute rather than `align="center"`. | yes                          |

## Locally checking

I have an Intel Mac and I was able to locally reproduce the HTML5 issues
with the following command using *the development version of R*:

``` r
rcmdcheck::rcmdcheck(env = c("_R_CHECK_RD_VALIDATE_RD2HTML_" = "TRUE"))
```

Note that I needed a relatively new version of the HTML tool `tidy`,
which I got with:

    brew install tidy-html5

To install the development version of R, I used
[rig](https://github.com/r-lib/rig).

    rig add devel
    # Note that on Intel Mac I needed to use this instead because the default
    # devel version rig downloads wasn't new enough:
    # rig add https://files.r-hub.io/macos/R-devel.pkg

    rig default 4.3

You will need to install all the packages required to check your
package, so open RStudio to your package directory and run:

``` r
pak::pak()
```

pak is installed automatically by rig and this will install all the
packages required to check with rcmdcheck.

## Specifics about the workflow

Note that the workflow is a little slow, due to having to install
pdflatex. It also errors on all NOTEs, rather than just on WARNINGs and
ERRORs (to catch the HTML issue), so you might get new failures that you
previously weren’t aware of.

-   This check is only in the development version of R, so `r-version`
    is set to `"devel"` and we are using `ubuntu-latest` as the OS.

-   `_R_CHECK_RD_VALIDATE_RD2HTML_=TRUE` is the environment variable
    that is set to trigger this check. This check only generates a NOTE,
    so to force rcmdcheck to fail here we set `error-on: '"note"'`.

-   To check the HTML manual, we have to actually build it. This is
    typically not done by default, so there are a few things we have to
    do:

    -   `args: '"--as-cran"'` is set to change the default away from
        `c("--no-manual", "--as-cran")'`

    -   `build_args: 'character()'` is set to change the default away
        from `build_args: '"--no-manual"'`

    -   pdflatex is installed to build the PDF version of the manual. It
        seems like you have to build both the PDF and HTML version of
        the manual, there is no way to attempt to build just one of
        them.

    -   tidy is installed to check the HTML version of the manual.
