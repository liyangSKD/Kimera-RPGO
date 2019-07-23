RobustPGO
======================================
This is a work in progress repository for robust backend optimization. Many features are still under developing and changing on a daily basis.

## Dependencies

*[GTSAM](https://github.com/borglab/gtsam)*
(Note that the BUILD_WITH_MARCH_NATIVE flag caused me some problems on a particular machine. )

Clone GTSAM to your preferred location:
```bash
git clone git@github.com:borglab/gtsam.git
```

Build
```bash
cd gtsam
mkdir build
cd build
cmake .. -DGTSAM_POSE3_EXPMAP=ON -DGTSAM_ROT3_EXPMAP=ON
sudo make install
```
**Note:**
The reason why we need EXPMAP is for the correct calculation of Jacobians.
Enabling this and the `#define SLOW_BUT_CORRECT_BETWEENFACTOR` in LaserLoopCLosure.h are both important. Otherwise the default are some identity approximations for rotations, which works for some cases but fails for cases with manual loop closures, or artifacts. Note that `sudo make check` will partially fail because some unittests assume the EXPMAP flags to be off.

## Build
```bash
git clone git@github.com:MIT-SPARK/RobustPGO.git
cd RobustPGO
mkdir build
cd build
cmake ..
make
```

## Usage
This repository can be used as an optimization backend. A sample setup looks something like below. The default solver is LM.
```cpp
// Set up
// set up RobustPGO solver
RobustSolverParams params;
params.setPcm3DParams(0.0, 10.0, Verbosity::QUIET);
// Verbosity levels are QUIET, UPDATE, and, VERBOSE in order of increasing number of messages (the default is UPDATE)
// For 2D params.setPcm2DParams(0.0, 10.0); Have been tested

// To use GaussNewton instead of LM: params.solver = Solver::GN;

std::unique_ptr<RobustSolver> pgo = std::make_unique<RobustSolver>(params);
//...
//...

// Run
pgo->update(new_factor, new_values);
```
This can also be used as a standalone experimental tool. A read g2o function can be found in examples.
```
# for 2D:
./RpgoReadG2o 2d <g2o-file> <odom-check-threshold> <pcm-threshold> <optional:folder-to-save-g2o> <optional:v to toggle verbosity>

# for 3D
./RpgoReadG2o 3d <g2o-file> <odom-check-threshold> <pcm-threshold> <optional:folder-to-save-g2o> <optional:v to toggle verbosity>
```

Example, do `./RpgoReadG2o 3d /home/user/Desktop/in.g2o 1.0 1.0 /home/user/Desktop/out.g2o v`