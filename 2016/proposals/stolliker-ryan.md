# Ryan Stolliker: Result Aggregation Server for Software Carpentry

## Abstract

During Software Carpentry's workshops which teach computing skills to the greater scientific community, learners use Python scripts to test that they have installed the programming environment and other dependencies correctly. Currently, this information is known only to the participants and cannot be used or tracked by the organizers. Thus, the goal of this project is to enable the collection of this installation data and to store it for future use and analysis.

## Technical Details

### Server

All server management will be written in Python. The server itself will be implemented using [bottle](http://bottlepy.org/docs/dev/index.html), a simple web framework with functionality to parse forms and JSON requests and responses. 

####Interactions

* `/first/`
  * The first installation script will send platform, system, release, implementation, version information here in JSON or forms format. The server will send back a 200 response.
* `/second/`
  * The second installation script will send platform, system, release, implementation, version and dependency data here in JSON or forms format. The server will send back a 200 response.
* `/data/`
  * A request to this URL will recieve JSON from the server which contains all information from the database for analysis in some future application.

###Database

SQLite will be used as the database to store the results of the installation scripts. SQLAlchemy will be used to manage the database and all Python interaction with the database. If database migration is needed in the future, Alembic will be used.

####Proposed Schema

```
CREATE TABLE System (
    SID INTEGER PRIMARY KEY AUTOINCREMENT,
    plat VARCHAR(30) NOT NULL,
    sys VARCHAR(10) NOT NULL,
    rel VARCHAR(10) NOT NULL,
    implementation VARCHAR(10) NOT NULL,
    version VARCHAR(6) NOT NULL
);
```
`System` is the table that holds data about the machine the script is running on. `SID` is an atuomatically-generated identifier assigned each time data is entered into the server. `Plat` is the system information provided by calling `platform.platform`. `Sys` is the operating system provided by `platform.system`. `Rel` is the version of the operating system provided by `platform.release`. `Implementation` is the Python distribution (CPython, PyPy, etc.) provided by `platform.python_implementation`. `Version` is the Python version provided by `platform.python_version`. The size of each `varchar` may be different in the actual database depending on the maximum length of the string returned by the platform functions.

```
CREATE TABLE Dependency (
    SID INTEGER PRIMARY KEY REFERENCES System (SID),
    success BOOLEAN NOT NULL
);
```
Each test in the [second test script](https://github.com/wking/swc-setup-installation-test/blob/master/swc-installation-test-2.py) will have its own table in the database, so `Dependency` is meant to be a placeholder for the name of the dependency that will also be the name of the table. This data is associated with `System` information by the `SID`. `Success` is a boolean which represents whether the specific test was passed. If a workshop does not test for a specific dependency, then there will not an entry for that dependency associated with the results from those machines. If more dependencies are added in the future, then a new table would have to be created, but no existing tables would have to be altered.

### Updates to Test Scripts

The existing installation testing scripts, already written in Python, will be updated to send data to the server. Because the point of the scripts is to determine whether Python is configured correctly, the changes should not rely on any third party libraries and be compatible with both Python 2 and 3 so that even if a system fails a test, it will still be able to send the results. Data will be sent using the http.client and urllib.parse libraries. Before sending, the user will be prompted to decide whether they actually want to send the information. System data will be collected using the platform library, which is included in Python's standard library, specifically:

* general platform information
  * `platform.platform()`
* Python implementation
  * `platform.python_implementation()`
* Python version
  * `platform.python_version()`
* Operating system information
  * `platform.uname()`
* Dependency test results
  * The [second test script](https://github.com/wking/swc-setup-installation-test/blob/master/swc-installation-test-2.py) tests that various libraries and programs are properly installed on the system, and this information will be sent to the server.

## Schedule of Deliverables

### May 25th -  June 7th

Create SQLite database schema using SQLAlchemy for high level manipulation of the database.

### June 8th - June 21th

Begin work on server, with focus on handling POST requests, interpreting them, and correctly using database functions to transfer received information into SQLite database.

### June 22nd - July 5th

Continue creating server, now focusing on aspects of how to retrieve information from database and serve it for analysis.

### July 6th - July 19th

Modify the [first test script](https://github.com/wking/swc-setup-installation-test/blob/master/swc-installation-test-1.py) to submit simple pass/fail data about the Python version to the server.

### July 20th - August 2nd

Modify the [second test script](https://github.com/wking/swc-setup-installation-test/blob/master/swc-installation-test-2.py) to submit data to the server on the platform and the pass/fail status of dependencies and tools.

### August 3rd - August 16th

Make a simple front end for easy viewing of data, as either a desktop or web application, which gets its information from the created API.

### August 17th - August 21th 19:00 UTC

Week to account for complications and delays, test on more machines and operating systems.

## Future works

If additional software dependencies are added to the installation scripts, then the server's handling of the request would need to change with it. Alembic would be used to implement any database migrations.

## Open Source Development Experience

This project with Software Carpentry will be my first experience with open source development. Some of my personal projects that I have done are [solving programming challenges](https://github.com/rstolliker/challenges) as well as [an iOS app](https://github.com/rstolliker/FirstApp).

## Academic Experience

I am a Computer Science student with a specialization in information at the University of California, Irvine. Some of my relevant coursework includes classes in database management and many levels of Python, including networking and object oriented design. Outside of university I have also taught myself HTML, CSS, and JavaScript.

## Why this project?

When I first came to college I was originally a Mechanical Engineering major. One of the first classes I took was a "programming for engineers" course that taught MATLAB for scientific analysis and graphing. My high school didn't offer any Computer Science courses so this was my first exposure to programming, but I was already hooked and soon I changed my major. Software Carpentry caught my attention because its goal of teaching computing skills to other scientific fields reminded me of my own journey into Computer Science.

## Appendix

[This is an example project that I did](https://github.com/rstolliker/APIexample) which includes an SQLite database, a server running bottle, and a client that collects some system information and sends it to the server.

[Here is the issue thread I opened.](https://github.com/numfocus/gsoc/issues/85)
###Contact Information

Email: ryan.stolliker@gmail.com

IRC: ryanCS on freenode

Pacific Time Zone (California)