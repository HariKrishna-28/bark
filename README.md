# Bark
Bark is supposed to be a small and easy-to-use library that uses PostgreSQL for collecting logs from multiple sources. It has a web server which can accept the logs using REST calls and for that reason we can run the server separately and can use client library (in progress) to send the logs to the server. But why are we writing yet another logger, aren't there many more already?

![Image](https://raw.githubusercontent.com/techrail/bark/main/_nocode/images/BARK.png)

## Introduction to the problem
When we start off with smaller projects, logging is not an issue. You either do a `fmt.Println` or a `log.Print` and view and search for text in text files. However, as the app or the organisation grows in size and start creating multiple services and start logging more things, logging becomes more and more problematic. Filtering inside a log file becomes more difficult. Co-relating logs to form a single, isolated flow of events becomes a problem too. Sorting and searching that information across multiple large log files and keeping track of line numbers becomes confusing and debugging with the help of logs becomes a nightmare.

Now, there are pretty great projects, both open and proprietary out there that make log collection, search and analytics on terabyte scale possible. However, there are a few problems that we can observer: 

- The more capable ones are either more costly, or more complex, or sometimes both. 
- Setting up a dedicated logging solution is not always easy or possible given various constraints like time and manpower, testability, locality of data, cost, required expertise for the selected solution etc.)
- Then comes the learning curve of querying - different solutions have different query languages and varying nuances between them all.  

So it takes time to install, configure and learn such solutions.

However, between the basic HTTP server and the enterprise scale, as we grow, we still need to be able to store, process, analyze and search through our logs.

# Aim of this project

![Image](https://raw.githubusercontent.com/techrail/bark/main/_nocode/images/where-bark-fits.png)

Bark aims to fill the gap that exists between simple file-based logging and a large log aggregation solution. Bark is: 

- **Easy to setup**: We use PostgreSQL for storing logs. It is cross-platform and installing it is easy. You probably already have it installed for your regular services anyway!
- **Easy to configure**: You just need to create a new database (or use an existing one), create a new table and start a server. That's it.
- **Easy (or nothing) to learn**: You probably already know SQL. You can use that knowledge to filter and analyse all your logs! There is nothing new to learn.

## Ease of purging irrelevant logs
When the time comes, you can search through your logs as needed. You can dispose off anything that's unnecessary in one way or the other. For example, with Bark, you can _easily_ do something like this : 

- _Delete logs that are more than 3 months old_: Useful for saving storage 
- _Delete all logs that are of `INFO` level_: Useful when you want to delete all logs which do not represent any kind of failure or danger.
- _Delete all logs that are either `INFO` or `WARNING` level and were sent by `pdf_printer` service between the dates: `2023-06-03` and `2023-06-09`_: Useful when you know you had a problem that caused enormous amounts of logs to be emitted but the problem is solved now!
- _Delete everything older than a month except the ones with code `ABCDEF`_: Useful when you wanna clear off old logs but leave some for looking deeper into an old problem.

All of these are very easily doable using SQL!

We want this service to be available as a web service which can be called by other services to send in the log entries. Bark also aims to include a library which can be included in your golang project and be called from within the project using just a function.
 
## Caveat
Bark is supposed to scale alongside you till a point. Bark uses PostgreSQL for storing logs and is thus pretty performant.

However, it is worth noting that PostgreSQL is not designed to store enormous amounts of time-series data. But PostgreSQL is just good enough for mid-size installations (say about a dozen services) that has not gone global yet. So bark is not aimed at becoming the most powerful and flexible log aggregation service but useful enough for searching and filtering logs.

# Language of Choice
It has to be written in golang for the sake of being great at handling incoming traffic bursts, lining up the logs and sending them in a load-controlled manner to PostgreSQL (because PostgreSQL is not optimized for that kinda stuff).

# Installation and Usage
### Prerequisite

- Go ([Golang](https://go.dev/)) version 1.20+
- PostgreSQL database version 12+

## Installation

Bark uses go modules. You can install bark using docker as well.

(To be expanded after some more work is done and we have released version 1.0)

# Setup

Following are the steps to set up the library on a machine after cloning the repository:
- Set the appropriate value for `DATABASE_URL` environment variable. 
The `DATABASE_URL` should be of the format `postgres://username:password@host:port/db?sslmode=disable`. For example: `export DATABASE_URL="postgres://vaibhav:mypassword@127.0.0.1:5432/log_db?sslmode=disable"`
- Navigate to the directory containing the `go.mod` file.
- Install the dependencies using the command `go get .`
- To create the required tables navigate to the `_nocode/db/migrations` folder. Copy SQL commands from all the `.up.sql`, and run them in the `psql` terminal. Or you can use a migration tool like [golang-migrate](https://github.com/golang-migrate/migrate)
- Run the bark server using the command `go run main.go`

To test if the library is up and running as expected, open a browser and navigate to the URL: [localhost:8080/hello/vaibhav](http://localhost:8080/hello/vaibhav)

You should see a text rendered on your browser saying `Hello, vaibhav!` 

# What is it NOT?
- **It is not a replacement for Plaintext logs** - Bark should be able to write to a plaintext log file in parallel to throwing items into Postgres. In case Bark server cannot write to the database, it will emit your log messages on the server's STDOUT.
- **It is not an APM** - We don't want to throw in Application uptime or Performance Monitoring. Bark is not supposed to be a monitoring solution at all.
- **It is not trying to replace any Terabyte-scale log aggregation service** - e.g. ELK Stack, NewRelic, DataDog etc. are dedicated more towards enterprise requirements and have capability to handle terabytes of logs. Bark does not aim to act as a replacement of such services. It aims to be the stepping stone between plaintext and terabyte-scale, enterprise-ready solutions.
- **It is not a CLI tool or a Web server at this point** - Bark, at this time does not offer a Web Service or a CLI tool to view, filter or tail your logs. For that you would have to run a query against PostgreSQL directly using your terminal or GUI tool of your choice (tailing on logs might not be possible though).

Social Media Preview and header image Photo by [Lora Ninova](https://unsplash.com/@lorannva?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) on [Unsplash](https://unsplash.com/photos/U86FnrpRR0k?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)
