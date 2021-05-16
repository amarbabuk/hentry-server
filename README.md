# Hentry

<p align="center">
  <img width="460"  src="https://raw.githubusercontent.com/YashKumarVerma/hentry-server/master/illustrations/hentry-logo.png?token=ADLB4KZZHZPB3MEDI3GRA63AVIDKI">
</p>

Hentry, is a hackathon sentry that allows organizers to provide a fair competing platform in the online events. Since plagiarism and re-use are the major problems in such submissions which ruins the spirit of hackathons,  It utilizes intelligent algorithms to calculate project entropy and snapshots of participants' projects in real-time and visualizes the same for the organizers as a live graph in a pleasant user interface.


## Components

- [Hentry Server](https://github.com/YashKumarVerma/hentry-server) : web server written in TypeScript to act as the service to expose data for hentry-dashboards. Provides routes which utilize Redis-JSON and Redis-TimesSeries to return data for business logic. Handle user creation, team creation, initial dashboard data fetch and polling updates for live graphs.
- [Hentry Client](https://github.com/YashKumarVerma/hentry-client) : CLI written in Golang utilizing go-routines for performance ⚡ which calculates project entropy and snapshots and emits them to hentry-feeder

- [Hentry Feeder](https://github.com/YashKumarVerma/hentry-feeder) : micro service written in GoLang utilizing go-routines for performance, exposes a simple HTTP server to accept requests from henry-client instances running on participants' devices and write data to Redis TimeSeries Database.
-  [Hentry Dashboard](https://github.com/YashKumarVerma/hentry-dashboard) : a responsive and dynamic single page application build using React and TailWind CSS, designed in a monochrome and minimal UI to focus on important data. Also provisions realtime graphs which render live feed of project status.


API Collection: [Here](https://documenter.getpostman.com/view/10043948/TzRLmqrE#intro)

## Architecture
![Project Architecture](https://raw.githubusercontent.com/YashKumarVerma/hentry-server/master/illustrations/map.png?token=ADLB4KYTBXCLY4N2QTW5J5TAVJJP2)

## Hentry Client

![https://i.imgur.com/hfATaxW.png](https://i.imgur.com/hfATaxW.png)
- **The Binary**
  - Can be compiled for any platform, any architecture as far as GoLang supports it. 
  - Generates unique signature for each device, which cannot be altered by changing configurations or be spoofed. So single machine cannot act as multiple devices.
  - Does **NOT** require admin privileges.
  - Device signatures are hardware independent, as they can be easily spoofed by VMs. MAC and BIOS settings are also ignored as they can be easily manipulated.
  - Automatically identifies the platform to display in the admin panel.
  - A list of pre-compiled binaries is available on [hentry-client.surge.sh](http://hentry-client.surge.sh/). It is however recommended to compile for yourself as we're in early testing phase.

- **The Interface**
  - Interactive command line interface with option to navigate using arrow keys and validation check indicators builtin. If the validation is about to fail, the terminal shows red with an error message and there's no need to work-up again and again.
  - Secure TeamID Input : Since team IDs are used to join a team, the interface masks the input with `*` to add a layer of security in the user interface.

- **Configuration**
  - To make it effortless for users to use the application, server credentials can be embedded into the binary itself by the organizers (single point configuration declarations), and the participants can directly run the same.
  - In case if there are changes in server deployments, or the organizers come up with alternative servers to relay the updates, a configuration file can be used to declare the endpoints.
  - The configuration file should be named **hentry.yaml** and be placed in the same directory as the binary sits.
  ```yml
    app:
      server: "https://some-fancy-server/api"
      feeder: "https://some-fancy-server/feed"
      debug: true

    ignore:
      - "public"
      - "cache"
      - "build"
      - "some-custom"
  ```
  - Restarting the client will first check if a configuration file is present. If its not, then display a message to ensure that the user knows, and go ahead to load the default configurations.
  - **Ignore Custom Directories** : since there can by multiple directories which contain auto-generated codebase, depending on the tech stack being used, therefore there is a provision to list all of them in the config file in the **ignore** section. hentry would then ignore those directories while calculating snapshots and entropy.

- **Functionality**
  - Allows user to create a team of people working together in an event / competition.
  - Allows user to join an existing team 
  - Allows user to register their device 
  - Thanks to unique device signatures, can identity old devices and avoid duplicate logins.
  - Walks through the project to calculate entropy and snapshot scores and feeds them to api servers.

- **Fancy Tech**
  - Written implementing go-routines for concurrency ⚡ and speed, utilizes minimal system resources, non blocking.
  - Recursively walks the the directory tree and creates a hashmap of the project in the memory.
  - Calculates the snapshot score and entropy of all the files in each iteration.
  - Snapshot score depends upon the file contents, length and size.
  - Entropy is a name given to diff-match score, which is basically the number of insertions and deletions required to move from one state to another. 
  - The entropy is calculated using the [golang port](https://pkg.go.dev/github.com/sergi/go-diff/diffmatchpatch) of [Neil Fraser's google-diff-match-patch](https://github.com/google/diff-match-patch) code 
  - Written in Golang, can be compiled to native binary for any operating system and architecture.
  
- **Debug Mode**
  - the **hentry,yaml** contains an option to enable the debug mode.
  - This mode is added to help developers debug the deployments. Once its turned on, the binary will log all API request its sending and receiving.
### Walkthrough

Launching without any configurations or admin rights:
![https://i.imgur.com/TVZFUHS.png](https://i.imgur.com/TVZFUHS.png)

Validation of input in CLI, alongwith live status display with ✔️ and ✖️ depending on input
![https://i.imgur.com/NsGGv2Q.png](https://i.imgur.com/NsGGv2Q.png)

![https://i.imgur.com/9bEHQaH.png](https://i.imgur.com/9bEHQaH.png)

Similar checks for Team Name
![https://i.imgur.com/T9JlE54.png](https://i.imgur.com/T9JlE54.png)

`hentry-client` automatically checks if device is already registered or not. Since my device was not registered, it asks me to register
![https://i.imgur.com/pwSJntM.png](https://i.imgur.com/pwSJntM.png)

Shows a success message when device is successfully registered.
![Device Registered](https://i.imgur.com/dgpnUnk.png)

Starts transmitting data to `hentry-server`
![Events being transmitted](https://i.imgur.com/zkOBFOP.png)

The above was a demo when user was not already in a team, what if wants to join a team and they're already registered?

Joining a team with team ID, note that the input is masked.
![Joining a Team](https://i.imgur.com/rozDbtd.png)

Thanks to unique device signatures, hentry client identifies that the device is already registered and automatically adds it to the said team, and starts data transmission
![Device attached to team](https://i.imgur.com/iOFqztN.png)

Lets play around with the config file. Note that `hentry-client` shows a list of directories that it ignores during startup, and what if the server configs change? To handle such cases, use the **hentry.yaml** file. I use the following `hentry-config` file.

![https://i.imgur.com/ngzg8e1.png](https://i.imgur.com/ngzg8e1.png)

Now when we run the `hentry-client`, we can see that the new configurations are loaded, and more directories are being ignored.
![https://i.imgur.com/zXs44zu.png](https://i.imgur.com/zXs44zu.png)

Also since the **debug** flag was set to true, the client now logs all the api calls it makes.
![https://i.imgur.com/aawruiB.png](https://i.imgur.com/aawruiB.png)

## Hentry Server

![https://i.imgur.com/TlCaJbc.png](https://i.imgur.com/TlCaJbc.png)

- Hentry Server is written in typescript with modern tooling to quickly prototype the application.
- The server is available on docker hub as yashkumarverma[](https://hub.docker.com/repository/docker/yashkumarverma/hentry-server)/hentry-server
- Connects to reddismod instance and ensures that connection with json and timeseries module is made.
- Provides routes for team formation, team joining, device registration, fetching all timeseries data and polling for updates in timeseries data.
- To run locally, run `yarn install` then `yarn start:dev` or use the docker image.
- Follows a uniform logging scheme to make it easier to debug.
- Supports keywords like `now` to fetch all data till the present timestamp.
- Divided into modules and services keep related codebase together and therefore make it easier to maintain.
- Configurations can be accessed in `config` directory.

![https://i.imgur.com/9HEwswE.png](https://i.imgur.com/9HEwswE.png)
![https://i.imgur.com/SqBM14v.png](https://i.imgur.com/SqBM14v.png)

## Hentry Feeder

![https://i.imgur.com/gXnIxAV.png](https://i.imgur.com/gXnIxAV.png)

- Hentry feeder is a tiny microservice, designed to run in clusters depending upon the number of participants on the system.
- Only purpose is to inject data received from the clients (hentry-client which can go upto thousands in number depending on event) into redis timeseries database.
- To compile manually, run `make build` or compile `internal/main.go` for your architecture.
- Also available as a docker container : [yashkumarverma/hentry-feeder](https://hub.docker.com/repository/docker/yashkumarverma/hentry-feeder)

## Hentry Dashboard

![https://i.imgur.com/AFGOWdM.png](https://i.imgur.com/AFGOWdM.png)
- main interface that is used by event organizers.
- clean, minimal ui
- option to search via team ID in landing page

![https://i.imgur.com/PebX7cQ.png](https://i.imgur.com/PebX7cQ.png)

- shows the friendly name of each device in team, and also their platforms.
![https://i.imgur.com/XOUdZYP.png](https://i.imgur.com/XOUdZYP.png)

- When a device a hovered 
![https://i.imgur.com/Nqp6ecN.png](https://i.imgur.com/Nqp6ecN.png)

- opening the details of any device shows a live graph of project snapshot and entropy.
- (refer video)


## Deployment
after running respective docker images, something like this should be accessible. or you can use the hosted docker images directly instead of building them manually.

The images are
- yashkumarverma/hentry-server
- yashkumarverma/hentry-feeder
- redismod
![https://i.imgur.com/aGDxRG1.png](https://i.imgur.com/aGDxRG1.png)
