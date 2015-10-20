[![Build Status](https://travis-ci.org/bast/pfunit-demo.svg?branch=master)](https://travis-ci.org/bast/pfunit-demo/builds)
[![Coverage Status](https://coveralls.io/repos/bast/pfunit-demo/badge.png?branch=master)](https://coveralls.io/r/bast/pfunit-demo?branch=master)


# pfunit-demo

Fortran unit test demo using [pFUnit](http://pfunit.sourceforge.net) deployed to
[Travis-CI](https://travis-ci.org/bast/pfunit-demo/builds) with test coverage
deployed to [Coveralls](https://coveralls.io/r/bast/pfunit-demo).

- [Build and test history](https://travis-ci.org/bast/pfunit-demo/builds)
- [Code coverage](https://coveralls.io/r/bast/pfunit-demo)
- Licensed under [BSD-3](../master/LICENSE)


## How to build this demo

```
git clone --recursive https://github.com/bast/pfunit-demo.git
cd pfunit-demo
mkdir build
cd build
cmake ..
make -j
make test
```

To get more output when tests fail:
```
CTEST_OUTPUT_ON_FAILURE=1 make test
```

## Contributors

- Guillaume Jacquenot (added possibility to use a local install for pFUnit)
