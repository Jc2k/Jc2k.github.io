---
layout: page
title: Projects
---

# Projects

# Active Projects

### Touchdown

[![Travis Status](https://img.shields.io/travis/yaybu/touchdown/master.svg)](https://travis-ci.org/yaybu/touchdown)
[![Appveyor status](https://img.shields.io/appveyor/ci/yaybu/touchdown/master.svg)](https://ci.appveyor.com/project/yaybu/touchdown)
[![Codecov](https://img.shields.io/codecov/c/github/yaybu/touchdown/master.svg)](https://codecov.io/github/yaybu/touchdown?ref=master)
[![Release](https://img.shields.io/pypi/v/touchdown.svg)](https://pypi.python.org/pypi/touchdown/)
[![Docs](https://img.shields.io/badge/docs-latest-green.svg)](http://docs.yaybu.com/projects/touchdown/en/latest/)
[![Started](https://img.shields.io/badge/started-sep_2014-green.svg)](http://github.com/yaybu/touchdown)

Unhappy with the status quo in cloud deployment automation I started Touchdown, a graph based approach for describing, deploying and then maintaining cloud infrastructure. Primarily deals with AWS. It started from the ideas of Yaybu's parallel lazily evaluation model with a declarative resource layer and stripped them bare.


### Fuselage

[![Travis Status](https://img.shields.io/travis/yaybu/fuselage/master.svg)](https://travis-ci.org/yaybu/fuselage)
[![Appveyor status](https://img.shields.io/appveyor/ci/yaybu/fuselage/master.svg)](https://ci.appveyor.com/project/yaybu/fuselage)
[![Codecov](https://img.shields.io/codecov/c/github/yaybu/fuselage/master.svg)](https://codecov.io/github/yaybu/fuselage?ref=master)
[![Release](https://img.shields.io/pypi/v/fuselage.svg)](https://pypi.python.org/pypi/fuselage/)
[![Docs](https://img.shields.io/badge/docs-latest-green.svg)](http://docs.yaybu.com/projects/fuselage/en/latest/)
[![Started](https://img.shields.io/badge/started-may_2014-green.svg)](http://github.com/yaybu/fuselage)

Fuselage is Yaybu but without a lazy evaulated DSL. It was created because the interface is really expressive and the code is really lightweight. I had lots of use cases where something like Chef was overkill, and had already written the code.


## Experiments

### libcloudcore

[![Travis Status](https://img.shields.io/travis/Jc2k/libcloudcore/master.svg)](https://travis-ci.org/Jc2k/libcloudcore)
[![Appveyor status](https://img.shields.io/appveyor/ci/Jc2k/libcloudcore/master.svg)](https://ci.appveyor.com/project/Jc2k/libcloudcore)
[![Codecov](https://img.shields.io/codecov/c/github/Jc2k/libcloudcore/master.svg)](https://codecov.io/github/Jc2k/libcloudcore?ref=master)
[![Started](https://img.shields.io/badge/started-jun_2015-green.svg)](http://github.com/Jc2k/libcloudcore)

This was an attempt to create a library like botocore for multiple cloud providers. It was an introspective look at where libcloud was and where botocore was and how adapting ideas from botocore could making libcloud more testable or maintainable and allow e.g. a twisted based API to be generated.
