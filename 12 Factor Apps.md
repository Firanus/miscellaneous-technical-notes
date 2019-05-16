# 12 Factor Apps

Methodology for building SaaS apps.

A piece of software built using the 12 factor app methodology should have a number of ideal properties.

* It should use a declarative format for setup automation.
* It should have a clean contract with the underlying operating system for maximum portability
* It should deploy on modern cloud platforms
* It should have minimal divergence between its development and production environments
* It should be able to (but might not necessarily) enable Continuous Deployment for maximum agility
* It should scale up with minimal effort or change.

## The 12 Factors

### 1. Codebase

> One codebase tracked in revision control, many deploys

Self-explanatory. Interestingly, if you have multiple codebases, you have a distributed system, not an app. Each piece of the distributed system should be treated as its own app.

Multiple apps sharing a common code base is a violation. Instead, factor shared code into libraries you can pull in via a dependency manager.

### 2. Dependencies

> Explicitly declare and isolate dependencies

In other words, use your programming language's package management system to package up code that needs to be shared across multiple functions. Your deployment package can then contain your code, and the dependencies it relies on.

### 3. Config

> Store config in the environment

Environment variables are key-value pairs of information that can be set at deploy or update time.

These can be used to do any number of things. The most crucial detail about them is that they remain seperate from the rest of your application. A good rule of thumb is whether you could make your entire codebase open source immediately without compromising security.

Also, they should not be tied to a specific environment (e.g. `dev`, `qa`, `prod` etc.)

### 4. Backing Services

> Treat backing services as attached resources

A backing service is any service the app consumes over the network as part of its normal operation. The key detail with 12 factor apps is that they *make no distinction between local and third party services*. Local databases and third party APIs, for instance are all accessed in analagous ways.

In serverless, you get this by default, as all services are already considered external services that you reference through an HTTP endpoint or a DNS name.

Connection strings and the like are passed in via environment variables.

### 5. Build, release, run

> Strictly seperate build and run stages

A codebase is transformed into a (non-dev) deploy through 3 stages:

1. The **build stage** is a transform which converts a code repo into an executable bundle known as a build. This includes fetching dependencies and then compiling binaries and assets.
2. The **release stage** takes the build produced by the build stage and combines it with the deploy’s current config. The resulting release is ready for immediate execution in the execution environment.
3. The **run stage** (also known as “runtime”) runs the app in the execution environment, by launching some set of the app’s processes against a selected release.

In practice, this principle can be adhered to by having a CI or CD pipeline. The build step of such a pipeline corresponds to the build stage above. Unit tests and linting also fall under this stage. The release stage is the deployment and testing. Finally, we promote that artefact to be in the run stage.

The key here is that all of this is managed by the pipeline. If any of these steps fail, we go back to square 1, fix the issue, and then go through the whole pipeline again. This means we don't inadvertantly tinker with our code before in the release and run stages.

### 6. Processes

> Execute the app as one or more stateless processes

The key word here is that all our apps should be stateless. Any state that you need to access or modify should be stored in a backing service and treated as an external resource as per item 4.

Serverless functions are stateless by definition, so you get this one for free.

### 7. Port Binding

> Export services via port binding

Sometimes, web apps are ececuted inside a webserver container that is injected at runtime. 12 Factor apps, in contrast, are completely self-contained. They export HTTP (or other services) as a service by binding to a port, and listening to requests coming in through that port.

This also means that one app can become the backing service for another app.

In serverless, there aren't ports. However, you do have event sources, which have 3 different invocation models:
* Synchronous - get an event, wait for a callback
* Asynchronous - get an event, don't wait for a callback
* Stream-based - where events are triggered by a streaming source

### 8. Concurrency

> Scale out via the process model

A computer, once run, can be represented by one or more processes. For example, PHP processes run as child processes of Apache, started on demand as needed by request volume. Java processes take the opposite approach, with the JVM providing one massive uberprocess that reserves a large block of system resources (CPU and memory) on startup, with concurrency managed internally via threads. In both cases, the running process(es) are only minimally visible to the developers of the app.

In a 12 factor app, *processes are first class citizens*. Developers can architect their app to handle diverse workloads by assigning each type of work to a particular process type. For example, HTTP requests might be handled by a web process, whereas long-running background threads could be handled by a worker process.

The key here is that an application must be able to span multiple processes running on multiple machines. One of the reasons, and greatest advantages of this is that it makes horizontal scaling trivial; you can just add another stateless process.

The reasoning here is very similar to the reasoning that underlies that of a microservices architecture.

In serverless, you get horizontal scaling automatically (up to a limit). You can do stuff like forking processes, but generally this is ill-advised.

### 9. Disposability

> Maximize roubstness with fast startup and graceful shutdown

The processes of a 12 factor app should be disposable, in other words, they *can be started or stopped at a moment's notice*. 

There are various degrees of complexity to account for here, to do with ensuring rapid startup, and how to handle different kinds of shut down (graceful vs. sudden death). Regardless, the expectation is that a process should be able to handle any kind of termination gracefully, and to start up quickly.

In serverless, we don't have a concept of shutdown to worry about. However, we do care about startup speed, which means we need concern ourselves with the things that affect it, including:
* Deployment package size
* Programming language used,
* Pre-handler code
* If the code is in a VPC or not

### 10. Dev/prod parity

> Keep development, staging, and production as similar as possible

The reasoning for this one is self-explanatory, and can be facilitated by a variety of other techniques, like the use of environment variables, configuration management, containerization and more.

### 11. Logs

> Treat logs as event streams

One of the important details here is that a 12 factor app never concerns itself with the routing or storage of its output stream. 

Instead, the environment the app is running in will handle routing and storing the logs as appropriate. In local development, this will be something analogous to an unbuffered `stdout`. In staging and production, logs might instead be routed to a log indexing and analysis system, or to a data warehousing system. The major cloud providers all provide tools for these purposes.

In serverless, you usually have a function passed in that allows logging, or the runtime automatically captures console logs.

### 12. Admin Processes

> Run admin/management tasks as one-off processes

One-off tasks developers want to perform, for instance a database migration or running of one-time scripts, should run through the same environment as the regular, long-running processes of an app.

This process should run against a release, use the same codebase and config as any other process run against that release, and should ship with application code to avoid synchronization issues. Finally, it should use the same dependency isolation techniques as all other process types.

In serverless, this concept doesn't exist, as everything is a one-off process.