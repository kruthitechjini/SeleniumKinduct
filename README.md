# VehicleFinanceModern-e2eTests
E2E/Acceptance tests for VFM Applications

To run the tests:
* Make sure your gradle properties file (usually `~/.gradle/gradle.properties`) has following properties, which are required to authenticate with nexus to download artifacts:

    ```
    nexusUsername=<your username>
    nexusPassword=<your password>
    ```
* If you do not have a `gradle.properties`file, create a text file with that name. Proceed to copy the aformentioned properties into it and enter your Nexus username and password. 

* Run from command-line:

    ```
    > git clone <repository>
    > ./gradlew clean test -Penv=<env> -Dwebdriver.chrome.driver=<path to chromedriver>
    ```

    where `<env>` can be `[local|dev|kahustage|kahuprod|bhphstage|bhphprod]`. Additional environments can be added in `config.groovy`. You'll also need to download appropriate [chromedriver][1] distribution and pass the path to it at command-line as shown, or put it in base project folder which is the default path.

* UI testing with Galen Framework (Layout): This package also includes UI/UX tests that validate the UI objects/styling etc. using [Galen Framework][2]. These tests can be run by using `galen` or `layout` group:

    ```
    > ./gradlew clean test -Penv=<env> -Pgroups=layout -Dwebdriver.chrome.driver=<path to chromedriver>
    ```
    The specs, which are written using [Galen's spec language][3] are located in: `src/test/resources/layout`. After test execution, results can be found in: `target/galen-html-reports/report.html`

**Customizing Test Executions**

* Filter by Groups: TestNG supports using groups to filter tests that need to be run. These groups can be passed in as a comma separated string:

    ```
    > ./gradlew clean test -Pgroups=smoke,login
    ```

    This will run all tests that belong to groups `smoke` or `login`.

    Groups can also be used to exclude certain tests. For example:

    ```
    > ./gradlew clean test -Pexgroups=slow
    ```

    Currently configured groups are:

    * `bvt`: (build verification tests) these tests can be run to quickly validate that a recently deployed application build is ready for further testing. This is a small subset of all tests and is supposed to provide a quick confidence in the build for further testing. The goal is to complete test execution for this group within 2 minutes. One example where this used is when a new build is deployed as part of CD pipeline, and these tests are run for each node:

        ```
        > ./gradlew clean test -Pgroups=bvt
        ```

        When all nodes are completed and to run a complete regression test against ELB, this group can be excluded:

        ```
        > ./gradlew clean test -Pexgroups=bvt,slow
        ```

    * `prodsmoke`: this group provides more coverage than BVTs and can be run after production deployment is complete to ensure that all the basic application functionality is working as expected. Running a full automated regression test against production ELB may not be feasible due to data limitations:

        ```
        > ./gradlew clean test -Pgroups=prodsmoke -Pexgroups=bvt
        ```

    * `slow`: these are tests that are known to be slow running and are intentionally excluded from CD runs. The goal is to kick them off outside the CD pipeline to ensure fast runs of pipeline, until these tests can be made to run faster.

* Retry on failure: You can also specify how many times each test will be retried if it fails. Default is 1.

    ```
    > ./gradlew clean test -PretryCount=2
    ```

* Parallel Execution: If you have multiple test classes, you can run them in parallel by specifying the number of threads to use. This can dramatically cut down on test execution time, but also add some thread safety issues as you have to define your data providers and setup/teardown methods carefully. Default is 1, so no parallelism.

    ```
    > ./gradlew clean test -PthreadCount=2
    ```

* Specify Base URL: If you want to override the base url for your environment (or target an individual host) you can pass in `baseUrl` as a parameter:

    ```
    > ./gradlew clean test -PbaseUrl=https://someinstance.spireon.com
    ```

* Logging: This project uses [logback][4] for logging. Default log level is WARN, and it can be overridden for debugging purposes:

    ```
    > ./gradlew clean test -Dlogging.rootLevel=DEBUG
    ```

These tests generate a html report located at `./build/reports/tests/test/index.html` and also a junit-style xml report that can be parsed by CI tools.

[1]: https://sites.google.com/a/chromium.org/chromedriver/downloads
[2]: http://galenframework.com/
[3]: http://galenframework.com/docs/reference-galen-spec-language-guide/
[4]: https://logback.qos.ch
