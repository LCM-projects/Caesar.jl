# Caesar.jl

[![Caesar](http://pkg.julialang.org/badges/Caesar_0.5.svg)](http://pkg.julialang.org/?pkg=Caesar&ver=0.5)
[![Caesar](http://pkg.julialang.org/badges/Caesar_0.6.svg)](http://pkg.julialang.org/?pkg=Caesar&ver=0.6)

A modern robotic toolkit for localization and mapping -- towards non-parametric / parametric navigation solutions.

| **Major Dependencies** |     **Status**     |    **Test Coverage**    |
|:-----------------------:|:------------------:|:------------------:|
| Caesar.jl | [![Build Status][build-img]][build-url] | [![codecov.io][cov-img]][cov-url] |
| [RoME.jl]([rome-url]) | [![Build Status][r-build-img]][r-build-url] | [![codecov.io][r-cov-img]][r-cov-url] |
| [IncrementalInference.jl]([iif-url]) | [![Build Status][iif-build-img]][iif-build-url] | [![codecov.io][iif-cov-img]][iif-cov-url] |
| [TransformUtils]([tf-url]) | [![Build Status][tf-build-img]][tf-build-url] | [![codecov.io][tf-cov-img]][tf-cov-url] |

This is a research and development driven project and intended to reduce the barrier of entry for Simultaneous Localization and Mapping (SLAM) systems. This [Julia](http://www.julialang.org/) package encompasses test cases and robot related software for multi-modal (multi-hypothesis) navigation and mapping solutions from various sensor data, made possible by [Multi-modal iSAM](http://frc.ri.cmu.edu/~kaess/pub/Fourie16iros.pdf).

Please see related packages, Robot Motion Estimate [RoME.jl]([rome-url]) and back-end solver [IncrementalInference.jl]([iif-url]).

## Examples

Intersection of ambiguous elevation angle from planar SONAR sensor:

<a href="http://vimeo.com/198237738" target="_blank"><img src="https://raw.githubusercontent.com/dehann/Caesar.jl/master/doc/imgs/rovasfm02.gif" alt="IMAGE ALT TEXT HERE" width="480" border="10" /></a>

Bi-modal belief

<a href="http://vimeo.com/198872855" target="_blank"><img src="https://raw.githubusercontent.com/dehann/Caesar.jl/master/doc/imgs/rovyaw90.gif" alt="IMAGE ALT TEXT HERE" width="480" border="10" /></a>

Multi-modal range only example:

<a href="http://vimeo.com/190052649" target="_blank"><img src="https://raw.githubusercontent.com/dehann/IncrementalInference.jl/master/doc/images/mmisamvid01.gif" alt="IMAGE ALT TEXT HERE" width="480" border="10" /></a>

Installation
------------

Requires via ```sudo apt-get install```, see [DrakeVisualizer.jl](https://github.com/rdeits/DrakeVisualizer.jl) for more details.

    libvtk5-qt4-dev python-vtk

Then install required Julia packages  

    julia> Pkg.add("Caesar")

Note that Database related packages will not be automatically installed. Please see section below for details.

Basic usage
-----------

Here is a basic example of using visualization and multi-core factor graph solving:

    addprocs(2)
    using Caesar, RoME, TransformUtils

    vc = startdefaultvisualization()
    defaultscene01!(vc)
    rovt = loadmodel(:rov)
    rovt(vc)

    initCov = 0.01*eye(6); [initCov[i,i] = 0.001 for i in 4:6];
    odoCov = 0.001*eye(6); [odoCov[i,i] = 0.001 for i in 4:6];
    rangecov, bearingcov = 3e-4, 2e-3

    fg = identitypose6fg(initCov=initCov)
    tf = SE3([0.0;0.7;0.0], Euler(pi/4,0.0,0.0) )
    addOdoFG!(fg, Pose3Pose3(tf, odoCov) )

    visualizeallposes!(vc, fg, drawlandms=false)

    addLinearArrayConstraint(fg, (4.0, 0.0), :x2, :l1, rangecov=rangecov,bearingcov=bearingcov)
    visualizeDensityMesh!(vc, fg, :l1, meshid=2)
    addLinearArrayConstraint(fg, (4.0, 0.0), :x1, :l1, rangecov=rangecov,bearingcov=bearingcov)

    solveandvisualize(fg, vc, drawlandms=false, densitymeshes=[:l1;:x2])


Major features
--------------

* Visualization through [MIT Director](https://github.com/rdeits/DrakeVisualizer.jl).

* A multicore SLAM server over tcp

    julia -p10 -e "using Caesar; tcpStringSLAMServer()"

* A multicore Bayes 2D feature tracking server over tcp

    julia -p10 -e "using Caesar; tcpStringBRTrackingServer()"

Database interaction layer
--------------------------

For using the solver on a DataBase layer (work in progress on centralized architecture ) see [CloudGraphs](https://github.com/GearsAD/CloudGraphs.jl.git),

Install [Neo4j](https://neo4j.com/) and add these packages to your Julia system

    Pkg.clone("https://github.com/GearsAD/Neo4j.jl.git")
    Pkg.clone("https://github.com/GearsAD/CloudGraphs.jl.git")

Modify CloudGraphs related lines from test/runtests.jl Ln 7 to true.

You should be able to rerun the four door test on both internal dictionaries and repeated on Neo4j DB

    Pkg.test("Caesar")

Go to your browser at localhost:7474 and run the Cypher query

    match (n) return n

to see current graph. You can delete the graph using the query

    match (n) detach delete n


Future targets
--------------

This is a work in progress package. Please file issues here as needed to help resolve problems for everyone!

Hybrid parametric and non-parametric optimization. Incrementalized update rules and properly marginalized 'forgetting' for sliding window type operation. We defined interprocess interface for multi-language front-end development.

[cov-img]: https://codecov.io/github/dehann/Caesar.jl/coverage.svg?branch=master
[cov-url]: https://codecov.io/github/dehann/Caesar.jl?branch=master
[build-img]: https://travis-ci.org/dehann/Caesar.jl.svg?branch=master
[build-url]: https://travis-ci.org/dehann/Caesar.jl

[rome-url]: http://www.github.com/dehann/RoME.jl

[r-cov-img]: https://codecov.io/github/dehann/RoME.jl/coverage.svg?branch=master
[r-cov-url]: https://codecov.io/github/dehann/RoME.jl?branch=master
[r-build-img]: https://travis-ci.org/dehann/RoME.jl.svg?branch=master
[r-build-url]: https://travis-ci.org/dehann/RoME.jl

[iif-cov-img]: https://codecov.io/github/dehann/IncrementalInference.jl/coverage.svg?branch=master
[iif-cov-url]: https://codecov.io/github/dehann/IncrementalInference.jl?branch=master
[iif-build-img]: https://travis-ci.org/dehann/IncrementalInference.jl.svg?branch=master
[iif-build-url]: https://travis-ci.org/dehann/IncrementalInference.jl

[iif-url]: http://www.github.com/dehann/IncrementalInference.jl

[tf-cov-img]: https://codecov.io/github/dehann/TransformUtils.jl/coverage.svg?branch=master
[tf-cov-url]: https://codecov.io/github/dehann/TransformUtils.jl?branch=master
[tf-build-img]: https://travis-ci.org/dehann/TransformUtils.jl.svg?branch=master
[tf-build-url]: https://travis-ci.org/dehann/TransformUtils.jl

[tf-url]: http://www.github.com/dehann/TransformUtils.jl
