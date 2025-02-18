[//]: # (title: Build Log)
[//]: # (auxiliary-id: Build Log)

A _build log_ is an enhanced console output of a build. It is represented by a structured list of the events which took place during the build. Generally, it includes entries on TeamCity-performed actions and the output of the processes launched during the build. TeamCity captures the processes output and stores it in an internal format that allows for hierarchical display.

## Viewing Build Log

The log of a specific build is available for browsing on the __[Build Results](build-results-page.md#Build+Log+Tab)__ page. 

The __Tree view__ is the most capable view provided in the web UI. By default, all messages are displayed. Using the _View_ drop-down menu, you can switch from all messages to viewing __errors__ separately, or you can choose __Important messages__ to see the log filtered by "error" and "warning" statuses. You can also use the "Verbose" view level and download a raw build log using the corresponding link.

>To switch to the dark theme in the build log, select the __Use console view__ option.

You can download a full build log in the textual form or as a `.zip` archive  from the Build Results page by clicking the _Download full build log_ link in the upper right corner. Alternatively, you can use the following URL: `http://teamcity:8111/httpAuth/downloadBuildLog.html?buildId=<id>`. It is also possible to download the build log as a `.zip` file using the corresponding link in the UI or via the following URL: `http://teamcity:8111/httpAuth/downloadBuildLog.html?buildId=&archived=true`.  
Note that if you authenticate in TeamCity via access token (for example, when sending an API request), you need to omit the `httpAuth` part of the URL and use `http://teamcity:8111/downloadBuildLog.html?buildId=<id>`.

## Customizing Log Output

You can modify the log output using [service messages](service-messages.md#Reporting+Messages+to+Build+Log). This includes grouping the log entries in blocks, handling parallel message flows, changing the severity of a log message, and so on.

<anchor name="BuildLog-LargeBuildLogsInspection"/>

## Build Log Size

It is recommended to keep the build log small and tune build scripts not to print too much into the output. Large build logs are hard to view in the browser and are loading TeamCity infrastructure piping build messages from the agent to the server while the build is running.

It is recommended to print into the output only the messages required to understand the build progress and build failures. The rest of the information should be streamed into a log file and the file should be published as a build artifact. A "good" build log size is megabytes at most.

### Partial Build Log Display

When opening large build logs, TeamCity displays a part of it to avoid browser hanging. You can view the full build log on clicking the corresponding link.

The display threshold is set to 5M characters by default and can be adjusted using the `teamcity.buildLog.sizeThreshold.chars` [internal property](server-startup-properties.md#TeamCity+Internal+Properties) (not applicable to running build logs and logs links with states (for example, direct links to messages).
{product="tc"}

The display threshold is set to 5M characters.
{product="tcc"}

## ANSI-style Coloring in Build Log

TeamCity build logs render clickable hyperlinks and support ANSI-style escape color codes by default: if your tool produces a colored console output, you will see in the build log in TeamCity. See the related feature request for the [full list of supported sequences](https://youtrack.jetbrains.com/issue/TW-23760#comment=27-1021150).

To disable the coloring, set the `teamcity.buildLog.ansiColoring.enabled=false` [internal property](server-startup-properties.md#TeamCity+Internal+Properties).
{product="tc"}
