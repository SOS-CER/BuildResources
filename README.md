# BuildResources
The BuildResources project contains the Ant configuration file [config/build.xml](config.build.xml) that will build jobs for CSC216 at NC State University.  The configuration file can be customized to support other build processes.

Run the Ant file with the command:

```
ant -DWORKSPACE=$WORKSPACE -DPROJECT_NAME=$PROJECT_NAME -DTS_PROJECT_NAME=$TS_PROJECT_NAME build_student ts_build unittest ts_unittest report
```

The following environment variables are set in the Jenkins job to pass to the Ant configuration:

  * `$PROJECT_NAME`: Name of top folder containing project submissions pushed to GitHub.
  * `$TS_PROJECT_NAME`: Name of folder on Jenkins Executor where the teaching staff test cases are stored
  
The build consists of the following targets:

  * `clean`: cleans the student's project
  * `ts_clean`: cleans up the teaching staff test information
  * `init`: sets up the folder structure for storing the results of all the various tools
  * `build_student`: compiles the students projects and tests - compiler errors result in failing the build
  * `ts_build`: compiles the teaching staff tests against the student's submission - if the teaching staff tests cannot compile, then the student submission does not meet the project design and the build fails
  * `pmd`: runs the [PMD](https://pmd.github.io/) static analysis tool on the student's code - we use a custom configuration that is available on request
  * `checkstyle`: runs the [Checkstyle](http://checkstyle.sourceforge.net/) static analysis tool on the student's code - we use a custom configuration that is available on request
  * `spotbugs`: runs the [SpotBugs](https://spotbugs.github.io/) static analysis tool on the student's code - we search for Scary and Scariest level notifications
  * `unittest`: runs the student's unit test suite against their solution instrumented to record coverage using [Jacoco](http://www.eclemma.org/jacoco/)
  * `checkTSTestingStatus`: checks to see if the teaching staff test can run (see below)
  * `ts_unittest`: runs the teaching staff unit tests against the student's solution
  * `report`: generates the report on static analysis and coverage for use by Jenkins

## Check Testing Status
We have created custom scripts to support checking that 1) students have written their own, passing unit tests with some level of statement coverage and 2) students are not using libraries that have been forbidden on the assignment.  

### checkTesting
The Ant `checkTesting` target runs the [`config/testingChecker.sh`](config/testingChecker.sh) script. The script checks that the student has written JUnit tests; that the tests do not have any static analysis notifications that identify poor testing practices like `assertTrue(true)` or test methods with no asserts; and that the student has a certain threshold of statement coverage on all non-UI classes.  The threshold of statement coverage is provided as an argument to the script.

### checkLibrary 
The Ant `checkLibrary` target runs the [`config/libraryChecker.sh`](config/libraryChecker.sh) script.  The script checks that there are no matches to the provided library strings (which include the full package to the library).  If a match is found in *any* of the project files (including documentation), the build is failed.