# Basic usage

* Needs to be compiled in the same directory as the sources. The `./piglit` binary expects it this way. So execute something like:
    ```
    cmake . -GNinja -DCMAKE_BUILD_TYPE=Debug
    ```
* running tests, example: `./piglit run quick_gl results/quick_gl-results`
* To view results execute `./piglit summary html summary/quick_gl-results results/quick_gl-results` and then open `summary/quick_gl-results/index.html` in a browser
* combining results *(i.e. to see regressions)*: `./piglit summary html summary/compare results/baseline results/current`
